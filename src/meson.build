inc_conf = include_directories('include')

cdata = configuration_data()

cdata.set_quoted('PACKAGE', meson.project_name())
cdata.set_quoted('VERSION', meson.project_version())
cdata.set_quoted('CONFDIR', dir_sysconf)
cdata.set_quoted('RUNSTATEDIR', dir_run_state)
cdata.set_quoted('LOCALSTATEDIR', dir_local_state)
cdata.set_quoted('SBINDIR', dir_sbin)

cdata.set('HAVE_' + os_name.to_upper(), 1)

foreach attr : ['malloc', 'nonnull', 'warn_unused_result']
  cc.has_function_attribute(attr)
endforeach

check_headers = [
  'arpa/inet.h',
  'arpa/nameser.h',
  'dirent.h',
  'getopt.h',
  'inttypes.h',
  'net/ethernet.h',
  'net/if.h',
  'net/if_arp.h',
  'net/if_types.h',
  'netdb.h',
  'netinet/icmp6.h',
  'netinet/if_ether.h',
  'netinet/in.h',
  'netinet/in6.h',
  'netinet/in_systm.h',
  'netinet/ip.h',
  'netinet/ip6.h',
  'netinet/ip_icmp.h',
  'netinet/tcp.h',
  'resolv.h',
  'stddef.h',
  'sys/file.h',
  'sys/ioctl.h',
  'sys/mman.h',
  'sys/param.h',
  'sys/resource.h',
  'sys/socket.h',
  'sys/stat.h',
  'sys/time.h',
  'sys/types.h',
  'sys/un.h',
  'sys/wait.h',
  'syslog.h',
  'termios.h',
]

check_functions = [
  'asprintf',
  'daemon',
  'fchmod',
  'fork',
  'gettimeofday',
  'mlockall',
  'putenv',
  'strsignal',
  'unsetenv',
]

check_types = [
  'struct arphdr',
  'struct ether_arp',
  'struct ether_header',
  'struct icmp',
  'struct icmp6_hdr',
  'struct ip',
  'struct ip6_hdr',
  'struct nd_neighbor_solicit',
  'struct nd_opt_hdr',
]

subdir('ed25519')
subdir('chacha-poly1305')

src_lib_tinc = [
  'conf.c',
  'dropin.c',
  'keys.c',
  'list.c',
  'names.c',
  'netutl.c',
  'script.c',
  'splay_tree.c',
  'sptps.c',
  'subnet_parse.c',
  'utils.c',
  'version.c',
  'xoshiro.c',
  'logger.c',
]

src_tinc = [
  'fsck.c',
  'ifconfig.c',
  'info.c',
  'invitation.c',
  'tincctl.c',
  'top.c',
]

src_tincd = [
  'address_cache.c',
  'autoconnect.c',
  'buffer.c',
  'compression.h',
  'conf_net.c',
  'connection.c',
  'control.c',
  'dummy_device.c',
  'edge.c',
  'event.c',
  'fd_device.c',
  'graph.c',
  'meta.c',
  'multicast_device.c',
  'net.c',
  'net_packet.c',
  'net_setup.c',
  'net_socket.c',
  'node.c',
  'process.c',
  'protocol.c',
  'protocol_auth.c',
  'protocol_edge.c',
  'protocol_key.c',
  'protocol_misc.c',
  'protocol_subnet.c',
  'raw_socket_device.c',
  'route.c',
  'subnet.c',
  'tincd.c',
]

cc_flags_tincd = cc_flags

deps_common = []
deps_tinc = []
deps_tincd = [cc.find_library('m', required: false)]

if os_name in ['linux', 'android']
  subdir('linux')
elif os_name.endswith('bsd') or os_name in ['dragonfly', 'darwin']
  subdir('bsd')
elif os_name == 'sunos'
  subdir('solaris')
elif os_name == 'windows'
  subdir('mingw')
endif

foreach h : check_headers
  if cc.has_header(h)
    cdata.set('HAVE_' + h.to_upper().underscorify(),
              1,
              description: '#include <' + h + '>')
  endif
endforeach

confdata = configuration_data()
confdata.merge_from(cdata)
configure_file(output: 'meson_config.h', configuration: confdata)

have_prefix = '''
  #include "@0@/src/meson_config.h"
  #include "@1@/have.h"
'''.format(build_root, meson.current_source_dir())

foreach f : check_functions
  if f == 'fork' and os_name == 'windows'
    message('MinGW does not have correct definition for fork()')
  else
    if cc.has_function(f, prefix: have_prefix, args: cc_defs)
      cdata.set('HAVE_' + f.to_upper(),
                1,
                description: 'function ' + f)
    endif
  endif
endforeach

if cc.has_function('res_init', prefix: '''
  #include <netinet/in.h>
  #include <resolv.h>
''', args: cc_defs)
  cdata.set('HAVE_DECL_RES_INIT', 1)
endif

foreach type : check_types
  if cc.has_type(type, prefix: have_prefix, args: cc_defs)
    name = 'HAVE_' + type.to_upper().underscorify()
    cdata.set(name, 1, description: type)
  endif
endforeach

if not cdata.has('HAVE_GETOPT_H') or not cc.has_function('getopt_long', prefix: have_prefix, args: cc_defs)
  src_lib_tinc += ['getopt.c', 'getopt1.c']
endif

if not opt_miniupnpc.disabled()
  dep_miniupnpc = dependency('miniupnpc', required: false, static: static)
  if not dep_miniupnpc.found()
    # No pkg-config files on MinGW
    dep_miniupnpc = cc.find_library('miniupnpc', required: opt_miniupnpc, static: static)
  endif
  if dep_miniupnpc.found()
    src_tincd += 'upnp.c'
    deps_tincd += [
      dependency('threads', static: static),
      dep_miniupnpc,
    ]
    if static
      cc_flags_tincd += '-DMINIUPNP_STATICLIB'
    endif
    cdata.set('HAVE_MINIUPNPC', 1)
  endif
endif

if opt_curses.auto() and os_name == 'windows'
  message('curses does not link under MinGW')
else
  # The meta-dependency covers more alternatives, but is only available in 0.54+
  curses_name = meson_version.version_compare('>=0.54') ? 'curses' : 'ncurses'
  dep_curses = dependency(curses_name, required: opt_curses, static: static)
  if dep_curses.found()
    cdata.set('HAVE_CURSES', 1)
    deps_tinc += dep_curses
  endif
endif

# Some distributions do not supply pkg-config files for readline
if opt_readline.auto() and os_name == 'windows'
  message('readline does not link under MinGW')
else
  dep_readline = dependency('readline', required: opt_readline, static: static)
  if not dep_readline.found()
    dep_readline = cc.find_library('readline', required: opt_readline, static: static)
  endif
  if dep_readline.found() and \
     cc.has_header('readline/readline.h', dependencies: dep_readline) and \
     cc.has_header('readline/history.h', dependencies: dep_readline)
    cdata.set('HAVE_READLINE', 1)
    deps_tinc += dep_readline
  endif
endif

dep_zlib = dependency('zlib',
                      required: opt_zlib,
                      static: static,
                      fallback: ['zlib', 'zlib_dep'])
if dep_zlib.found()
  cdata.set('HAVE_ZLIB', 1)
  deps_tincd += dep_zlib
endif

if not opt_lzo.disabled()
  dep_lzo = dependency('lzo2', required: false, static: static)
  if not dep_lzo.found()
    dep_lzo = cc.find_library('lzo2', required: opt_lzo, static: static)
  endif
  if not dep_lzo.found()
    dep_lzo = dependency('lzo2',
                         required: false,
                         static: static,
                         fallback: ['lzo2', 'lzo2_dep'])
  endif
  if dep_lzo.found()
    if dep_lzo.type_name() == 'internal' or cc.has_header('lzo/lzo1x.h', dependencies: dep_lzo)
      cdata.set('LZO1X_H', '<lzo/lzo1x.h>')
    elif cc.has_header('lzo1x.h', dependencies: dep_lzo)
      cdata.set('LZO1X_H', '<lzo1x.h>')
    else
      msg = 'lzo1x.h not found'
      if opt_lzo.auto()
        warning(msg)
      else
        error(msg)
      endif
    endif
    if cdata.has('LZO1X_H')
      cdata.set('HAVE_LZO', 1)
      deps_tincd += dep_lzo
    endif
  endif
endif

dep_lz4 = dependency('liblz4',
                     required: opt_lz4,
                     static: static,
                     fallback: ['lz4', 'liblz4_dep'])
if dep_lz4.found()
  deps_tincd += dep_lz4
  cdata.set('HAVE_LZ4', 1)
endif

dep_vde = dependency('vdeplug', required: opt_vde, static: static)
dep_dl = cc.find_library('dl', required: opt_vde)
if dep_vde.found() and dep_dl.found()
  cdata.set('ENABLE_VDE', 1)
  src_tincd += 'vde_device.c'
  deps_tincd += [dep_dl, dep_vde]
endif

if opt_jumbograms
  cdata.set('ENABLE_JUMBOGRAMS', 1)
endif

subdir(opt_crypto)

if opt_crypto != 'nolegacy'
  src_lib_crypto += ['cipher.c', 'digest.c']
endif

subdir('include')

lib_crypto = static_library(
  'tinc_crypto',
  sources: src_lib_crypto,
  dependencies: dep_crypto,
  implicit_include_directories: false,
  include_directories: inc_conf,
  build_by_default: false,
)

deps_lib_tinc = [deps_common, dep_crypto]

lib_tinc = static_library(
  'tinc',
  sources: src_lib_tinc,
  dependencies: deps_lib_tinc,
  link_with: [lib_ed25519, lib_chacha_poly, lib_crypto],
  implicit_include_directories: false,
  include_directories: inc_conf,
  build_by_default: false,
)

exe_tinc = executable(
  'tinc',
  sources: src_tinc,
  dependencies: [deps_lib_tinc, deps_tinc],
  link_with: lib_tinc,
  implicit_include_directories: false,
  include_directories: inc_conf,
  install: true,
  install_dir: dir_sbin,
)

exe_tincd = executable(
  'tincd',
  sources: src_tincd,
  dependencies: [deps_lib_tinc, deps_tincd],
  link_with: lib_tinc,
  c_args: cc_flags_tincd,
  implicit_include_directories: false,
  include_directories: inc_conf,
  install: true,
  install_dir: dir_sbin,
)

exe_sptps_test = executable(
  'sptps_test',
  sources: 'sptps_test.c',
  dependencies: deps_lib_tinc,
  link_with: lib_tinc,
  implicit_include_directories: false,
  include_directories: inc_conf,
  build_by_default: false,
)

exe_sptps_keypair = executable(
  'sptps_keypair',
  sources: 'sptps_keypair.c',
  dependencies: deps_lib_tinc,
  link_with: lib_tinc,
  implicit_include_directories: false,
  include_directories: inc_conf,
  build_by_default: false,
)

if os_name == 'linux'
  dep_rt = cc.find_library('rt')

  exe_sptps_speed = executable(
    'sptps_speed',
    sources: 'sptps_speed.c',
    dependencies: [deps_lib_tinc, dep_rt],
    link_with: lib_tinc,
    implicit_include_directories: false,
    include_directories: inc_conf,
    build_by_default: false,
  )

  benchmark('sptps_speed', exe_sptps_speed, timeout: 90)
endif

