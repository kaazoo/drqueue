#
# Copyright (C) 2001,2002,2003,2004,2005,2006,2007 Jorge Daza Garcia-Blanes
#
# This file is part of DrQueue
#
# DrQueue is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# DrQueue is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307
# USA
#
# $Id$
#

import sys
import glob
import os
import platform,re

def get_architecture(escape=False,underscore=True):
    machine = platform.machine()
    if underscore:
        machine = re.sub('\s','_',machine)
    if escape:
        machine = re.escape(machine)
    if not machine:
        machine = 'unknown'
    return machine

def wrapper_complete_command (cmdlist):
    kernel = os.uname()[0]
    arch = get_architecture()
    rlist = []
    for cmd in cmdlist:
        cmd = os.path.split(cmd)[1] # Removes any directory component
        rlist.append('.'.join([cmd,kernel,arch]))
    return rlist
     
def get_platform_name():
    name = sys.platform
    if name == 'win32':
       return 'cygwin'
    return name

def get_abspath_glob(path):
    pathlist=glob.glob(path)
    rlist=[]
    for file in pathlist:
      rlist.append(os.path.abspath(file))
    return rlist

def copy_with_clean(src_files,dest_files,dest_path,env):
    rlist = []
    for s,d in zip(src_files,dest_files):
        t = env.InstallAs(os.path.join(dest_path,d),s)
        #t = env.Command(os.path.join(dest_path,d),s,[ Copy("$TARGET","$SOURCE") ])
        #env.Clean(t,env.subst("$TARGET"))
        rlist.append(t)
    return rlist

def add_suffix_to_list(list,suffix):
    """Meant to add the .exe to list of binary files"""
    result = []
    for item in list:
        result.append("%s%s"%(item,suffix))
    return result

def get_bin_extension():
    """Returns '.exe' if system is win32"""
    result = ''
    if platform.system() == 'Windows':
        result = '.exe'
    return result

def get_pkgconfig_path():
    """Returns the path to what should be the right pkg-config for every OS"""
    result = r'pkg-config'
    if platform.system() == "Windows":
        result = r'c:\cygwin\bin\pkg-config.exe'
    return result

# Construction environment for the library (doesn't link with itself)
env_lib = Environment (ENV=os.environ)
#conf = Configure(env_lib)
# TODO: write configure tests
#env_lib = conf.Finish()

# Configuration options
opts = Options('scons.conf')
opts.AddOptions(PathOption('DESTDIR','Alternate root directory','',[]),
				PathOption('PREFIX','Directory to install under','/usr/local'),
				BoolOption('build_drqman','Build drqman',1))
opts.Update(env_lib)
opts.Save('scons.conf',env_lib)

Help(opts.GenerateHelpText(env_lib))

#conf = Configure(env_lib)
# TODO: write configure tests
#env_lib = conf.Finish()

# Installation paths
idir_prefix = os.path.normpath(os.path.join('${DESTDIR}','${PREFIX}','drqueue'))
idir_bin    = os.path.join(idir_prefix,'bin')
idir_etc    = os.path.join(idir_prefix,'etc')
idir_db     = os.path.join(idir_prefix,'db')
idir_logs   = os.path.join(idir_prefix,'logs')
idir_tmp    = os.path.join(idir_prefix,'tmp')
Export('env_lib idir_prefix idir_bin idir_etc idir_db idir_logs idir_tmp')

if sys.platform == 'win32':
    print "-> Win32 using Cygwin mode"
    Tool('mingw')(env_lib)

env_lib.Append (CPPPATH=['.','libdrqueue'])
env_lib.Append (CPPDEFINES = Split ('-DCOMM_REPORT -D_GNU_SOURCE ' \
                + '-D_NO_COMPUTER_POOL_SEMAPHORES -D_NO_COMPUTER_SEMAPHORES'),
                CPPFLAGS = Split ('-g -O0'),
                CXXFLAGS = ['-D__CPLUSPLUS',Split(env_lib.subst('$CCFLAGS')),
                            Split(env_lib.subst('$CPPDEFINES'))])

print "Platform is: ",sys.platform
if sys.platform == "linux2":
  env_lib.Append (CPPDEFINES = Split ('-D__LINUX'))
elif sys.platform == "darwin":
  env_lib.Append (CPPDEFINES = Split ('-D__OSX'))
elif sys.platform == "irix6":
  env_lib.Append (CPPDEFINES = Split ('-D__IRIX'))
  env_lib['CC'] = 'c99'
elif sys.platform == "cygwin":
  env_lib.Append (CPPDEFINES = Split ('-D__CYGWIN'))
  os.environ['PKG_CONFIG_PATH'] = 'C:\GTK\lib\pkgconfig'
elif sys.platform == "win32":
  env_lib.Append (CPPDEFINES = Split ('-D__CYGWIN'))
else:
  print "Unknown platform: %s"%(sys.platform,)
  exit (1)

# Base construction environment that links with the library
env = env_lib.Copy()
env.Append (LIBS = ['drqueue'],LIBPATH = ['libdrqueue'])

#
# libdrqueue.a
#
libdrqueue_src = get_abspath_glob(os.path.join('libdrqueue','*.c'))
libdrqueue = env_lib.Library ('libdrqueue/drqueue', libdrqueue_src)
Default (libdrqueue)

#
# drqman
#
def build_drqman():
    result = []
    if env_lib['build_drqman']:
        print "Building drqman"
        drqman_c = glob.glob (os.path.join('drqman','*.c'))
        env_gtkstuff = env.Copy ()
        env_gtkstuff.ParseConfig ('pkg-config --cflags --libs gtk+-2.0')
        drqman = env_gtkstuff.Program (os.path.join('drqman','drqman'),drqman_c)
        result.append(os.path.join('drqman','drqman'))
        Default (drqman)
    else:
        print "Not building drqman"
    return result

#
# Main
#
master = env.Program ('master.c')
slave = env.Program ('slave.c')
main_list = [ 'master', 'slave' ] + build_drqman()
Default (master)
Default (slave)

## Build python modules
#Export ('env','env_lib','libdrqueue','libdrqueue_src')
#SConscript(['python/SConscript'])

#
# Tools
#
cmdline_tools = [ 'jobfinfo','jobinfo','compinfo','requeue','cfgreader',
                  'cjob','blockhost','sendjob' ]
cpp_tools = [ 'sendjob' ]
ctools = {}
for tool in cmdline_tools:
    if tool not in cpp_tools:
        ctools[tool] = env.Program (tool+'.c')
    else:
        ctools[tool ]= env.Program (tool+'.cpp')
    Default(ctools[tool])

install_base = idir_prefix
bin_list = main_list + cmdline_tools
wrapped_bin_list = wrapper_complete_command (bin_list)
bin_list = add_suffix_to_list(bin_list,get_bin_extension())
wrapped_bin_list = add_suffix_to_list(wrapped_bin_list,get_bin_extension())
wrapped_bin_copies = copy_with_clean(bin_list,wrapped_bin_list,idir_bin,env)

etc_files = glob.glob(os.path.join('etc','*'))
copy_with_clean(etc_files,etc_files,idir_prefix,env)
bin_files = glob.glob(os.path.join('bin','*'))
copy_with_clean(bin_files,bin_files,idir_prefix,env)
perm_logs = env.Command (idir_logs,[],[Mkdir("$TARGET"),Chmod("$TARGET",0777)])
env.Clean(perm_logs,idir_logs)
perm_tmp = env.Command (idir_tmp,[],[Mkdir("$TARGET"),Chmod("$TARGET",0777)])
env.Clean(perm_tmp,idir_tmp)
perm_db = env.Command (idir_db,[],[Mkdir("$TARGET"),Chmod("$TARGET",0777)])
env.Clean(perm_db,idir_db)
#env.Depends (install_base,perm_logs)
#env.Depends (install_base,perm_tmp)
#env.Depends (install_base,perm_db)
env.Alias('install-wrapped-bin',wrapped_bin_copies)
env.Alias('install-bin',idir_bin)
env.Alias('install-etc',idir_etc)
env.Alias('install-tmp',perm_tmp)
env.Alias('install-db',perm_db)
env.Alias('install-logs',perm_logs)
env.Alias('install',['install-wrapped-bin','install-bin','install-etc','install-db','install-logs','install-tmp'])
