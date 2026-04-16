add_library(Hikari SHARED
        FunctionCallObfuscate.cpp
        CryptoUtils.cpp
        BogusControlFlow.cpp
        SubstituteImpl.cpp
        Substitution.cpp
        Flattening.cpp
        Utils.cpp
        SplitBasicBlocks.cpp
        AntiClassDump.cpp
        AntiDebugging.cpp
        AntiHooking.cpp
        StringEncryption.cpp
        IndirectBranch.cpp
        FunctionWrapper.cpp
        ConstantEncryption.cpp
        Obfuscation.cpp
        )
target_include_directories(Hikari PRIVATE ${CMAKE_SOURCE_DIR}/obfuscation)

add_dependencies(Hikari intrinsics_gen LLVMLinker)

if(WIN32 AND NOT MINGW)
  # On Windows (MSVC/clang-cl), link LLVM static libraries explicitly.
  llvm_map_components_to_libnames(llvm_libs core support irreader linker passes)
  target_link_libraries(Hikari PRIVATE ${llvm_libs})
elseif(APPLE)
  # On macOS, allow undefined LLVM symbols to be resolved at runtime from the
  # host process (e.g. clang). This avoids RTTI symbol resolution issues with
  # the pre-built LLVM static libraries (built with -fno-rtti).
  target_link_options(Hikari PRIVATE -undefined dynamic_lookup)
endif()
# On Linux, undefined symbols in a shared library are resolved at runtime from
# the host process, so no explicit LLVM linking is needed.

if (MINGW)
    set_target_properties(Hikari PROPERTIES
        LINK_FLAGS "-static -static-libgcc -Wl,-Bstatic,--whole-archive -lwinpthread -lstdc++ -Wl,--no-whole-archive -Wl,-Bdynamic"
    )
endif()
execute_process(
        COMMAND git log -1 --format=%H
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
        OUTPUT_VARIABLE HIKARI_GIT_COMMIT_HASH
        OUTPUT_STRIP_TRAILING_WHITESPACE
)
target_compile_definitions(Hikari PRIVATE "-DGIT_COMMIT_HASH=\"${HIKARI_GIT_COMMIT_HASH}\"")
