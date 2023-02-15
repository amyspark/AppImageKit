# src/build-runtime.cmake
# abandon all hope, all ye who enter here

# must not include -flto in the following flags, otherwise the data sections will be stripped out
runtime_cflags = [
    '-ffunction-sections',
    '-fdata-sections',
    '-fno-lto',
    '-DGIT_COMMIT="@0@"'.format(git_commit),
]
# must not include -Wl,--gc-sections in the following flags, otherwise the data sections will be stripped out
runtime_ldflags = [
    '-s',
    '-ffunction-sections',
    '-fdata-sections',
]

if get_option('appimagekit_runtime_enable_setproctitle')
    runtime_cflags += ['-DENABLE_SETPROCTITLE']
endif()

# objcopy requires actual files for creating new sections to populate the new section
# therefore, we generate 3 suitable files containing blank bytes in the right sizes
_16_blank_bytes = custom_target(
    '16_blank_bytes',
    command: ['dd', 'if=/dev/zero', 'bs=1','count=16', 'of=@OUTPUT@'],
    output: '16_blank_bytes',
)
_1024_blank_bytes = custom_target(
    '1024_blank_bytes',
    command: ['dd', 'if=/dev/zero', 'bs=1','count=1024', 'of=@OUTPUT@'],
    output: '1024_blank_bytes',
)
_8192_blank_bytes = custom_target(
    '8192_blank_bytes',
    command: ['dd', 'if=/dev/zero', 'bs=1','count=8192', 'of=@OUTPUT@'],
    output: '8192_blank_bytes',
)

runtime_0 = static_library(
    'runtime.0',
    files('runtime.c',),
    c_args: runtime_cflags,
)

runtime_1 = custom_target(
    'runtime.1',
    command: ['objcopy', '--add-section', '.digest_md5=@0@'.format(_16_blank_bytes.full_path()), '--set-section-flags', '.digest_md5=noload,readonly', '--add-section', '.upd_info=@0@'.format(_1024_blank_bytes.full_path()), '--set-section-flags', '.upd_info=noload,readonly', '--add-section' '.sig_key=@0@'.format(_8192_blank_bytes.full_path()), '--set-section-flags', '.sig_key=noload,readonly',
        runtime_0.extract_objects('runtime.c'),
        'runtime.1.o'
    ],
    depends: [_16_blank_bytes, _1024_blank_bytes, _8192_blank_bytes],
    output: 'runtime.1.o'
)

runtime = add_executable(
    'runtime',
    files('notify.c',),
    objects: runtime_1,
    link_language: 'c',
    dependencies: [libsquashfuse],
    link_with: [libappimage_shared],
)

if get_option('appimagekit_embed_magic_bytes')
    embed_magic_bytes_in_file = find_program('embed-magic-bytes-in-file.sh')

    runtime_1 = custom_target(
        'runtime',
        command: [
            embed_magic_bytes_in_file, '@OUTPUT@'
        ]
        output: runtime.full_path(),
        depends: runtime,
    )
else
    runtime_1 = runtime
endif

xxd = find_program('xxd')

runtime_embed = custom_target(
    'runtime_embed_header',
    command: [xxd, '-i', '@INPUT@', '@OUTPUT@'],
    input: runtime_1,
    output: 'runtime_embed.c',
)

# this is the normal CMakeLists...

appimagetool_sources = files(
    'appimagetool.c',
    'appimagetool_sign.c',
    'binreloc.c',
    # runtime_embed
)

common_compile_args = [
    '-D_FILE_OFFSET_BITS=64',
    '-DGIT_COMMIT="@0@"'.format(git_commit),
    '-DENABLE_BINRELOC',
]

appimage_compile_args = common_compile_args + [
    '-DBUILD_NUMBER="@0@"'.format(build_number),
    '-DBUILD_DATE="@0@"'.format(date),
]

appimage_deps = [
    libsquashfuse,
    libappimage,
    libgcrypt,
    libgpgme,
]

appimage_libs = [libappimage_shared,]

has_squashfuse_dlopen = compiler.compiles('''
        #include "squashfuse_dlopen.h"
        int main() { return 0; }
    ''',
    dependencies: libsquashfuse,
)

if has_squashfuse_dlopen
    appimage_compile_args += ['-DHAS_SQUASHFUSE_DLOPEN=1']
else
    # Add libfuseprivate.a since the pkg-config module doesn't include it
    libfuseprivate_lib = compiler.find_library('fuseprivate', static: true)
    libfuse_lib = compiler.find_library('fuse', static: true)
    appimage_libs += [libfuseprivate_lib, libfuse_lib]
    # Add the compressor libraries too.
    libzlib = dependency('zlib', static: true)
    lz4 = dependency('lz4', static: true)
    zstd = dependency('zstd', static: true)
    lzo2 = dependency('lzo2', static: true)
    appimage_libs += [libzlib, lz4, zstd, lzo2]
endif

if auxiliary_files_destination != ''
    message('Installing auxiliary files in path: @0@'.format(auxiliary_files_destination))
    appimage_compile_args += [
        '-DAUXILIARY_FILES_DESTINATION="@0@"'.format(auxiliary_files_destination)
    ]
endif

appimagetool_module = declare_dependency(
    compile_args: appimage_compile_args,
    dependencies: appimage_deps,
    link_with: appimage_libs,
)

appimagetool = executable(
    'appimagetool',
    appimagetool_sources,
    objects: runtime_embed,
    dependencies: appimagetool_module,
    install: true,
    install_tag: 'appimagetool',
)

apprun_module = declare_dependency(
    compile_args: common_compile_args,
)

apprun = executable(
    'AppRun',
    files('AppRun.c',),
    dependencies: apprun_module,
    install: true,
    install_tag: 'appimagetool',
)

validate = executable(
    'validate',
    files('validate.c',),
    dependencies: [libglib, libssl],
    link_with: [libappimage_shared],
    install: true,
)

digest = executable(
    'digest',
    files('digest.c',),
    dependencies: [libglib, libssl],
    link_with: [libappimage_shared],
    install: true,
)

digest_md5 = executable(
    'digest_md5',
    files('digest_md5.c',),
    build_by_default: false,
    dependencies: [libglib],
    link_with: [libappimage_shared],
)
