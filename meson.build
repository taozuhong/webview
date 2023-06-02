project ('webview', ['c', 'cpp'], 
         version: '1.0.0',
         default_options: ['cpp_std=c++17', 'build.cpp_std=c++17'])

ABI_MAJOR = '1'
compiler_c = meson.get_compiler('c')
compiler_cpp = meson.get_compiler('cpp')

# Setup commonly used options
prefix = get_option('prefix')
libdir = get_option('libdir')
datadir = get_option('datadir')
vapidir = join_paths(datadir, 'vala', 'vapi')

# platform specific dependenies
webview_dependencies = []
webview_includes = [include_directories('.')]
host_os = host_machine.system()
if (host_os == 'windows') or (host_os == 'cygwin')
  version_webview2='1.0.1774.30'
  cmd_result = run_command('curl', '-sSLO', 'https://dist.nuget.org/win-x86-commandline/latest/nuget.exe', '--output-dir', meson.global_build_root(), check: true)
  if (0 == cmd_result.returncode())
    run_command(meson.global_build_root() + '/nuget.exe', 'install', 'Microsoft.Web.Webview2', '-Version', version_webview2, '-OutputDirectory', meson.global_build_root(), check: true)
  endif

  webview2_include = meson.global_build_root() + '/Microsoft.Web.WebView2.' + version_webview2 + '/build/native/include'
  add_project_arguments('-I' + webview2_include, language: 'cpp')
  add_project_arguments('-DWEBVIEW_API=__declspec(dllexport)', language: 'cpp')
  add_project_link_arguments(['-lversion', '-lshlwapi'], language: 'cpp')
elif host_os == 'linux'
  webview_dependencies += dependency('gtk+-3.0', version: '>= 3.24')
  webview_dependencies += dependency('webkit2gtk-4.0', version: '>= 2.0')
elif host_os == 'darwin'
    webview_dependencies += dependency('gtk+-3.0', version: '>= 3.24')
    webview_dependencies += dependency('webkit2gtk-4.0', version: '>= 2.0')
endif

sources_webview = [
    'webview.cc'
]

headers_webview = [
    'webview.h',
]

libwebview = library(
    meson.project_name() + '-@0@'.format(ABI_MAJOR),
    sources_webview + headers_webview,
    dependencies: webview_dependencies,
    include_directories: webview_includes,
    install: true,
)

install_headers(headers_webview,
                subdir: 'libwebview-@0@'.format(ABI_MAJOR))

pkgconfig = import('pkgconfig')
pkgconfig.generate(libwebview,
         name: meson.project_name() + '-@0@'.format(ABI_MAJOR),
  description: 'Tiny cross-platform webview library',
     requires: webview_dependencies,
      version: meson.project_version(),
      subdirs: meson.project_name() + '-@0@'.format(ABI_MAJOR),
)

generate_gir = true
generate_vapi = true
if generate_gir
  gnome = import('gnome')
  webview_gir = gnome.generate_gir(libwebview,
                sources: sources_webview + headers_webview,
              nsversion: ABI_MAJOR,
              namespace: 'Webview',
        export_packages: 'Webview-@0@'.format(ABI_MAJOR),
                 header: 'webview.h',
          symbol_prefix: 'webview',
      identifier_prefix: 'webview',
              link_with: libwebview,
    include_directories: webview_includes + [webview2_include],
                install: true,
  )

  if generate_vapi
    webview_vapi = gnome.generate_vapi('libwebview-@0@'.format(ABI_MAJOR),
          sources: webview_gir.get(0),
          install: true,
      install_dir: vapidir,
         packages: [],
        vapi_dirs: [vapidir],
    )
  endif
endif


libwebview_dep = declare_dependency(
    link_with: libwebview,
    dependencies: webview_dependencies,
    include_directories: webview_includes
)

if get_option('debug')
    sources_example_c = [
    'examples/basic.c',
    ]

    sources_example_cpp = [
    'examples/basic.cc',
    ]

    executable(meson.project_name() + '_c',
    sources_example_c,
    include_directories: webview_includes,
    dependencies: webview_dependencies,
    link_with: libwebview,
    install: true,
    win_subsystem: 'windows'
    )

    executable(meson.project_name() + '_cpp',
    sources_example_cpp,
    include_directories: webview_includes,
    dependencies: webview_dependencies,
    link_with: libwebview,
    install: true,
    win_subsystem: 'windows'
    )
endif