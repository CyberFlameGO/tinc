project('tinc', 'c',
  version: run_command('./src/git_tag.sh', check: true).stdout().strip(),
  license: 'GPL-2.0-or-later',
  meson_version: '>=0.51',
  default_options: [
    'c_std=c11',
    'warning_level=3',
    'buildtype=debugoptimized',
  ],
)

dir_run_state = get_option('runstatedir')
opt_crypto = get_option('crypto')
opt_curses = get_option('curses')
opt_docs = get_option('docs')
opt_harden = get_option('hardening')
opt_jumbograms = get_option('jumbograms')
opt_lz4 = get_option('lz4')
opt_lzo = get_option('lzo')
opt_miniupnpc = get_option('miniupnpc')
opt_readline = get_option('readline')
opt_static = get_option('static')
opt_systemd = get_option('systemd')
opt_tests = get_option('tests')
opt_tunemu = get_option('tunemu')
opt_uml = get_option('uml')
opt_vde = get_option('vde')
opt_zlib = get_option('zlib')

meson_version = meson.version()

cc = meson.get_compiler('c')
os_name = host_machine.system()
cc_name = cc.get_id()

cc_defs = ['-D_GNU_SOURCE']
cc_flags = [cc_defs]
ld_flags = []

if opt_static.auto()
  static = os_name == 'windows'
else
  static = opt_static.enabled()
endif

if static
  ld_flags += '-static'
endif

if opt_harden
  cc_flags += [
    '-D_FORTIFY_SOURCE=2',
    '-fwrapv',
    '-fno-strict-overflow',
    '-Wreturn-type',
    '-Wold-style-definition',
    '-Wmissing-declarations',
    '-Wmissing-prototypes',
    '-Wstrict-prototypes',
    '-Wredundant-decls',
    '-Wbad-function-cast',
    '-Wwrite-strings',
    '-fdiagnostics-show-option',
    '-fstrict-aliasing',
    '-Wmissing-noreturn',
  ]
  if cc_name == 'clang'
    cc_flags += '-Qunused-arguments'
  endif
  ld_flags += ['-Wl,-z,relro', '-Wl,-z,now']
  if os_name == 'windows'
    ld_flags += ['-Wl,--dynamicbase', '-Wl,--nxcompat']
  endif
endif

cc_flags = cc.get_supported_arguments(cc_flags)
ld_flags = cc.get_supported_link_arguments(ld_flags)

add_project_arguments(cc_flags, language: 'c')
add_project_link_arguments(ld_flags, language: 'c')

build_root = meson.current_build_dir()
src_root = meson.current_source_dir()

prefix = get_option('prefix')
dir_bin = prefix / get_option('bindir')
dir_data = prefix / get_option('datadir')
dir_info = prefix / get_option('infodir')
dir_lib = prefix / get_option('libdir')
dir_local_state = prefix / get_option('localstatedir')
dir_locale = prefix / get_option('localedir')
dir_man = prefix / get_option('mandir')
dir_sbin = prefix / get_option('sbindir')
dir_sysconf = prefix / get_option('sysconfdir')

if dir_run_state == ''
  dir_run_state = dir_local_state / 'run'
endif

if not opt_docs.disabled()
  subdir('doc')
endif

subdir('src')

if not opt_tests.disabled()
  subdir('test')
endif

subdir('bash_completion.d')

if os_name == 'linux' and not opt_systemd.disabled()
  subdir('systemd')
endif

prog_reformat = find_program('astyle', native: true, required: false)
if prog_reformat.found()
  run_target('reformat', command: [
    prog_reformat,
    '--options=@SOURCE_ROOT@/.astylerc',
    '@SOURCE_ROOT@/*.c', '@SOURCE_ROOT@/*.h',
  ])
endif

