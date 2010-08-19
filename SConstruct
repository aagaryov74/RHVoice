import sys
import os
import os.path

Execute(Mkdir("build"))
SConsignFile(os.path.join("build","scons"))
if sys.platform=="win32":
    toolset=["mingw"]
else:
    toolset=["default"]
env=Environment(tools=toolset,builddir="#build",package_name="RHVoice",package_version="0.1")
env["CPPPATH"]=["."]
env["LIBPATH"]=[]
env["CPPDEFINES"]=[("PACKAGE",env.subst(r'\"$package_name\"')),("VERSION",env.subst(r'\"$package_version\"'))]
var_cache=os.path.join("build","user.conf")
args={"DESTDIR":""}
args.update(ARGUMENTS)
vars=Variables(var_cache,args)
if env["PLATFORM"]!="win32":
    vars.Add("prefix","Installation prefix","/usr/local")
    vars.Add("DESTDIR","Support for staged installation","")
vars.Add("CC","C compiler",env.get("CC"))
vars.Add("FLAGS","Additional compiler/linker flags")
vars.Add("CCFLAGS","C compiler flags")
vars.Add("LINKFLAGS","Linker flags")
vars.Add(EnumVariable("enable_source_generation","Enable regeneration of C sources from Scheme sources","no",["yes","no"],ignorecase=1))
vars.Update(env)
vars.Save(var_cache,env)
Help("Type 'scons' to build the package.\n")
if env["PLATFORM"]!="win32":
    Help("Then type 'scons install' to install it.\n")
    Help("Type 'scons --clean install' to uninstall the software.\n")
Help("You may use the following configuration variables:\n")
Help(vars.GenerateHelpText(env))
flags=env.get("FLAGS")
if flags:
    env.MergeFlags(flags)
if env["PLATFORM"]!="win32":
    env["bindir"]=os.path.join(env["prefix"],"bin")
    env["datadir"]=os.path.join(env["prefix"],"share",env["package_name"])
    env["voxdir"]=os.path.join(env["datadir"],"voice")
    inst=Install(env["DESTDIR"]+env["voxdir"],Glob(os.path.join("data","voice","*.*[f123]")))
    Alias("install",inst)
    AddPostAction(inst,Chmod("$TARGET",0644))
    Clean("install",env["datadir"])
if GetOption("clean"):
    enable_config=False
elif GetOption("help"):
    enable_config=False
else:
    enable_config=True
if enable_config:
    conf=env.Configure(conf_dir=os.path.join("build","configure_tests"),log_file=os.path.join("build","configure.log"))
    if not conf.CheckCC():
        print "The C compiler is not working"
        exit(1)
    conf.CheckLib("m",autoadd=1)
    has_flite_h=conf.CheckCHeader("flite.h")
    if (not has_flite_h) and (env["PLATFORM"]!="win32"):
        for prefix in ["/usr/local","/usr"]:
            flite_cpppath=os.path.join(prefix,"include","flite")
            if os.path.isdir(flite_cpppath):
                if not GetOption("silent"):
                    print "trying to search in %s" % flite_cpppath
                env.Prepend(CPPPATH=flite_cpppath)
                has_flite_h=conf.CheckCHeader("flite.h")
                break
    if not has_flite_h:
        print "Error: cannot find flite.h"
        exit(1)
    has_libflite=conf.CheckLibWithHeader("flite","flite.h","C",call="flite_init();",autoadd=0)
    if not has_libflite:
        if hasattr(os,"uname"):
            if os.uname()[0]=="Linux":
                if not GetOption("silent"):
                    print "Perhaps this version of flite depends on alsa"
                if conf.CheckLib("asound",autoadd=1):
                    has_libflite=conf.CheckLibWithHeader("flite","flite.h","C",call="flite_init();",autoadd=0)
    if not has_libflite:
        print "error: cannot link with the flite library"
        exit(1)
    env.PrependUnique(LIBS="flite")
    if not conf.CheckLib("flite","cst_utf8_explode",autoadd=0):
        print "This version of flite is too old, please install flite 1.4 or later"
        exit(1)
    if not conf.CheckLib("flite_cmulex",autoadd=0):
        print "error: cannot link with the flite_cmulex library"
        exit(1)
    env.PrependUnique(LIBS="flite_cmulex")
    has_libunistring=conf.CheckLibWithHeader("unistring","unistr.h","C",call="u8_strchr(\"a\",'a');",autoadd=0)
    if not has_libunistring:
        if not GetOption("silent"):
            print "Perhaps libunistring requires libiconv on this platform"
        if conf.CheckLib("iconv",autoadd=1):
            has_libunistring=conf.CheckLibWithHeader("unistring","unistr.h","C",call="u8_strchr(\"a\",'a');",autoadd=0)
    if not has_libunistring:
        print "Error: cannot link with libunistring"
        exit(1)
    env.PrependUnique(LIBS="unistring")
    env=conf.Finish()

Export("env")
SConscript(dirs="src")
