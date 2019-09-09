# This file is part of libmicrodns.
#
# Copyright © 2019 VideoLabs SAS
#
# Author: Mathieu Duponchelle <mathieu@centricular.com>
#
#########################################################################
# libmicrodns is released under LGPLv2.1 (or later) and is also available
# under a commercial license.
#########################################################################
# This program is free software; you can redistribute it and/or modify it
# under the terms of the GNU Lesser General Public License as published by
# the Free Software Foundation; either version 2.1 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with this program; if not, write to the Free Software Foundation,
# Inc., 51 Franklin Street, Fifth Floor, Boston MA 02110-1301, USA.

project('microDNS', ['c'],
  version : '0.1.0',
  meson_version : '>= 0.49.0',
  default_options : ['warning_level=2',
                     'buildtype=release',
                     'b_ndebug=if-release'])

mdns_version = meson.project_version()

ver_arr = mdns_version.split('.')

mdns_major_version = ver_arr[0]
mdns_minor_version = ver_arr[1]
mdns_micro_version = ver_arr[2]

cc = meson.get_compiler('c')

warning_flags = [
    '-Wall',
    '-Wsign-compare',
    '-Wextra',
    '-Wstrict-aliasing',
    '-Wstrict-overflow',
    '-Wformat=2',
    '-Wno-unused-parameter',
    '-pipe',
    '-Wcast-align',
    '-Wpointer-arith',
    '-Wmissing-prototypes',
    '-Wwrite-strings',
    '-Wlogical-op',
]

add_project_arguments(cc.get_supported_arguments(warning_flags), language: 'c')

cdata = configuration_data()

deps = []

host_system = host_machine.system()

if host_system == 'windows'
    deps += [cc.find_library('ws2_32')]
    deps += [cc.find_library('iphlpapi')]
endif

inet_ntop_src = '''
#ifdef _WIN32
#include <ws2tcpip.h>
#include <windows.h>
# if _WIN32_WINNT < 0x600
#  error Needs vista+
# endif
#else
#include <sys/socket.h>
#include <arpa/inet.h>
#endif
int main() {}
'''

if cc.links(inet_ntop_src, dependencies: deps)
    cdata.set('HAVE_INET_NTOP', 1)
endif

poll_src = '''
#include <stddef.h>
#ifdef _WIN32
#include <winsock2.h>
#include <windows.h>
# if _WIN32_WINNT < 0x600
#  error Needs vista+
# endif
#else
#include <poll.h>
#endif
int main() {}
'''

if cc.links(poll_src, dependencies: deps)
    cdata.set('HAVE_POLL', 1)
endif

configure_file(output : 'config.h', configuration : cdata)

c_args = ['-DHAVE_CONFIG_H']

libcompat_sources = [
    'compat/compat.c',
    'compat/inet.c',
    'compat/poll.c',
]

incdirs = include_directories('src', 'compat')

libcompat = static_library('compat', libcompat_sources,
    include_directories: incdirs,
    dependencies: deps,
    c_args: c_args
)

libmicrodns_sources = [
    'src/mdns.c',
    'src/rr.c'
]

libmicrodns = library('microdns', libmicrodns_sources,
    include_directories: incdirs,
    link_with: libcompat,
    dependencies: deps,
    c_args: c_args,
)

examples_kwargs = {
    'include_directories': incdirs,
    'link_with': libmicrodns,
}

executable('listen', 'examples/main.c', kwargs: examples_kwargs)
executable('announce', 'examples/announce.c', kwargs: examples_kwargs)

pkg_cdata = configuration_data()

pkg_cdata.set('prefix', join_paths(get_option('prefix')))
pkg_cdata.set('exec_prefix', '${prefix}')
pkg_cdata.set('libdir', '${prefix}/@0@'.format(get_option('libdir')))
pkg_cdata.set('includedir', '${prefix}/@0@'.format(get_option('includedir')))
pkg_cdata.set('VERSION', mdns_version)
pkg_cdata.set('LIBSOCKET', host_system == 'windows' ? '-lws2_32 -liphlpapi': '')
pkg_cdata.set('abs_top_builddir', meson.current_build_dir())
pkg_cdata.set('abs_top_srcdir', meson.current_source_dir())

configure_file(
    input: 'src/microdns.pc.in',
    output: 'microdns.pc',
    configuration: pkg_cdata
)

configure_file(
    input: 'src/microdns-uninstalled.pc.in',
    output: 'microdns-uninstalled.pc',
    configuration: pkg_cdata
)

mdns_dep = declare_dependency(link_with : libmicrodns,
    include_directories : incdirs,
    dependencies: deps,
)

doc_cdata = configuration_data()

doc_cdata.set('PACKAGE_NAME', meson.project_name())
doc_cdata.set('VERSION', mdns_version)
doc_cdata.set('abs_top_srcdir', meson.current_source_dir())

configure_file(
    input: 'doc/Doxyfile.in',
    output: 'Doxyfile',
    configuration: doc_cdata
)
