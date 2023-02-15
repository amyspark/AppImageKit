project('appimagekit', 'c',
    version: '1-alpha',
    meson_version: '>=0.54.0',
    default_options: ['warning_level=3', 'c_std=gnu99']
)

git_commit = run_command('git', '-C', meson.current_source_dir(), 'rev-parse', '--short', 'HEAD',
    capture: true
).stdout().strip()

date = run_command('date', '-u', '"+%Y-%m-%d %H:%M:%S %Z"',
    capture: true,
    env: {
        'LC_ALL': 'C',
    }
).stdout().strip()

compiler = meson.get_compiler('c')

# cmake/dependencies.cmake
fuse = dependency('libfuse')
libssl = dependency('openssl', required: false, disabler: true)
libgio = dependency('gio-2.0', version: '>=2.40', required: false)
libsquashfuse = dependency('squashfuse', static: true)
# undeclared via import lib/libappimage
libappimage = dependency('libappimage', static: true)
libappimage_shared = compiler.find_library('appimage_shared', static: true)
libglib = dependency(
    'glib-2.0',
    version: '>=2.40',
    required: false, disabler: true,
)
# src/CMakeLists.txt
libgpgme = dependency('gpgme', required: false, static: true)
if not libgpgme.found()
    # Before ubuntu 20.04 libgpgme didn't have a pkg-config file
    # so relying on modules is actually a lie
    libgpgme_lib = compiler.find_library('gpgme', static: true)
    libassuan_lib = compiler.find_library('assuan', static: true)
    libgpgerror_lib = compiler.find_library('gpg-error', static: true)
    libgpgme = declare_dependency(
        link_with: [libgpgme_lib, libassuan_lib, libgpgerror_lib]
    )
endif
libgcrypt = dependency('gcrypt', required: false, static: true)
if not libgpgme.found()
    # Before ubuntu 20.04 libgpgme didn't have a pkg-config file
    # so relying on modules is actually a lie
    libgcrypt_lib = compiler.find_library('gcrypt', static: true)
    libgpgerror_lib = compiler.find_library('gpg-error', static: true)
    libgcrypt = declare_dependency(
        link_with: [libgcrypt_lib, libgpgerror_lib]
    )
endif
# undeclared via src/build-runtime.cmake
objcopy = find_program('objcopy')

subdir('src')
