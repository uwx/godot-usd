#!/usr/bin/env python

import glob
import os


def _expand(path):
    return os.path.abspath(os.path.expanduser(path))


godot_cpp_path = _expand(ARGUMENTS.get("godot_cpp_path", os.environ.get("GODOT_CPP_PATH", "/Users/miguel/cvs/Terrain3D/godot-cpp")))
usd_sdk_path = _expand(ARGUMENTS.get("usd_sdk_path", os.environ.get("USD_SDK_PATH", "/Users/miguel/cvs/usd/install")))
tbb_sdk_path = _expand(ARGUMENTS.get("tbb_sdk_path", os.environ.get("TBB_SDK_PATH", "/opt/homebrew/opt/tbb")))

if not os.path.isdir(godot_cpp_path):
    raise RuntimeError("godot-cpp checkout not found. Set GODOT_CPP_PATH or pass godot_cpp_path=/path/to/godot-cpp.")

if not os.path.isdir(os.path.join(usd_sdk_path, "include")) or not os.path.isdir(os.path.join(usd_sdk_path, "lib")):
    raise RuntimeError("OpenUSD SDK not found. Set USD_SDK_PATH to an install prefix with include/ and lib/.")

env = SConscript(os.path.join(godot_cpp_path, "SConstruct"))

env.Append(CPPPATH=["src"])
env.AppendUnique(CPPPATH=[os.path.join(usd_sdk_path, "include")])
env.AppendUnique(LIBPATH=[os.path.join(usd_sdk_path, "lib")])

if os.path.isdir(os.path.join(tbb_sdk_path, "include")):
    env.AppendUnique(CPPPATH=[os.path.join(tbb_sdk_path, "include")])

if os.path.isdir(os.path.join(tbb_sdk_path, "lib")):
    env.AppendUnique(LIBPATH=[os.path.join(tbb_sdk_path, "lib")])

if env["platform"] == "web" or (env["platform"] == "ios" and ARGUMENTS.get("static_usd", "no") == "yes"):
    usd_archives = sorted(glob.glob(os.path.join(usd_sdk_path, "lib", "libusd_*.a")))
    if not usd_archives:
        raise RuntimeError("static_usd=yes requires a static OpenUSD SDK.")
    env.AppendUnique(CPPDEFINES=["PXR_STATIC"])
    if env["platform"] == "web":
        env.Append(LINKFLAGS=["-Wl,--whole-archive"] + usd_archives + ["-Wl,--no-whole-archive", "--use-port=zlib"])
    else:
        env.Append(LINKFLAGS=[f"-Wl,-force_load,{archive}" for archive in usd_archives])
        env.AppendUnique(LINKFLAGS=["-framework", "Foundation"])
        env.AppendUnique(LIBS=["objc"])
    env.AppendUnique(LIBS=["tbb"])
else:
    env.AppendUnique(LIBS=[
        "usd_usdUtils",
        "usd_usdLux",
        "usd_usdShade",
        "usd_usdSkel",
        "usd_usdGeom",
        "usd_usd",
        "usd_sdf",
        "usd_tf",
        "usd_vt",
        "usd_gf",
        "usd_arch",
        "tbb",
    ])

if env["platform"] == "android":
    env.AppendUnique(LINKFLAGS=["-nostdlib++"])
    env.AppendUnique(LIBS=["c++_shared"])

if env["platform"] in ("linux", "macos", "ios"):
    if ARGUMENTS.get("bundle_dependencies", "no") == "yes":
        origin = "@loader_path" if env["platform"] in ("macos", "ios") else "'$$ORIGIN'"
        env.AppendUnique(LINKFLAGS=[f"-Wl,-rpath,{origin}"])
    else:
        env.AppendUnique(LINKFLAGS=[f"-Wl,-rpath,{os.path.join(usd_sdk_path, 'lib')}"])
        if os.path.isdir(os.path.join(tbb_sdk_path, "lib")):
            env.AppendUnique(LINKFLAGS=[f"-Wl,-rpath,{os.path.join(tbb_sdk_path, 'lib')}"])

sources = Glob("src/*.cpp")

library = env.SharedLibrary(
    "project/bin/{}-{}/libgodot_usd{}{}".format(env["platform"], env["arch"], env["suffix"], env["SHLIBSUFFIX"]),
    source=sources,
)

env.NoCache(library)
Default(library)
