project('libvmaf', ['c', 'cpp'],
    version : '3.0.0',
    default_options : ['c_std=c11',
                       'cpp_std=c++11',
                       'warning_level=2',
                       'buildtype=release',
                       'default_library=both',
                      ],
    meson_version: '>= 0.56.1')

vmaf_soname_version       = '3.0.0'
vmaf_api_version_array    = vmaf_soname_version.split('.')
vmaf_api_version_major    = vmaf_api_version_array[0]
vmaf_api_version_minor    = vmaf_api_version_array[1]
vmaf_api_version_revision = vmaf_api_version_array[2]

libvmaf_src_root = meson.current_source_dir()
cc = meson.get_compiler('c')
libvmaf_inc = include_directories(['include'])

# Arguments in test_args will be used even on feature tests
test_args = ['-D_GNU_SOURCE', '-D_USE_MATH_DEFINES']
add_project_arguments(test_args, language: ['c', 'cpp'])

# Header checks
if cc.has_header('unistd.h', args: test_args)
    add_project_arguments('-DHAVE_UNISTD_H', language: ['c', 'cpp'])
endif
if cc.has_header('malloc.h', args: test_args)
    add_project_arguments('-DHAVE_MALLOC_H', language: ['c', 'cpp'])
endif
if cc.has_header('alloca.h', args: test_args)
    add_project_arguments('-DHAVE_ALLOCA_H', language: ['c', 'cpp'])
endif
if cc.has_header('direct.h', args: test_args)
    add_project_arguments('-DHAVE_DIRECT_H', language: ['c', 'cpp'])
endif
if cc.has_header('corecrt_io.h', args: test_args)
    add_project_arguments('-DHAVE_CORECRT_IO_H', language: ['c', 'cpp'])
endif

stdatomic_dependency = []
if cc.has_header('stdatomic.h', args: test_args)
    if cc.check_header('stdatomic.h', args: test_args)
        stdatomic_dependency = declare_dependency(compile_args: '-DHAVE_STDATOMIC_H')
    elif cc.get_id() == 'msvc'
        # If stdatomic.h exists but isn't usable under MSVC it may be because recent (VS2022) MSVC
        # has added support for lock-free atomics where (sizeof(T) <= sizeof(intptr_t))
        # Note that __STDC_NO_ATOMICS__ remains defined due to lack of locking atomics
        test_args += '/experimental:c11atomics'

        if not cc.check_header('stdatomic.h', args: test_args)
            error('stdatomic.h exists but could not be made usable under MSVC with /experimental:c11atomics')
        endif

        add_project_arguments(['/experimental:c11atomics', '-DHAVE_STDATOMIC_H'], language: 'c')
    else
        error('stdatomic.h was found but is not usable for some reason; check whether the compiler needs extension flags')
    endif
elif cc.get_id() == 'msvc'
    # we have a custom replacement for MSVC before /experimental:c11atomics was added
    stdatomic_dependency = declare_dependency(
        compile_args : '-DHAVE_STDATOMIC_H',
        include_directories : include_directories('src/compat/msvc'),
    )
elif cc.compiles('''int main() { int v = 0; return __atomic_fetch_add(&v, 1, __ATOMIC_SEQ_CST); }''',
                 name : 'GCC-style atomics', args : test_args)
    stdatomic_dependency = declare_dependency(
        compile_args : '-DHAVE_STDATOMIC_H',
        include_directories : include_directories('src/compat/gcc'),
    )
else
    error('Atomics not supported')
endif

if cc.check_header('getopt.h', args: test_args)
    getopt_dependency = declare_dependency(compile_args : '-DHAVE_GETOPT_H')
else
    getopt_dependency = declare_dependency(
        compile_args : '-DHAVE_GETOPT_H',
        sources: ['src/compat/getopt/getopt.c'],
        include_directories : include_directories('src/compat/getopt'),
    )
endif

subdir('include')
subdir('src')
subdir('tools')
subdir('doc')
subdir('test')
