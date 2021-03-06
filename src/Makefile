# Example Makefile

# -- Dirs -- #
CC 			=gcc
EDIR    =exengine
LDIR		=lib
BDIR		=build
ODIR 		=obj
IDIR    =inc
EDATA   =exengine
DATA    =data
_IDIRS  =lib inc lib/physfs exengine
IDIRS   =$(patsubst %,-I%/,$(_IDIRS))

# -- Flags -- #
FLAGS 	=-g -lm -ldl -Wall -Wno-unused -Wno-uninitialized -lstdc++ \
-lGL -lSDL2 -lopenal -I. $(IDIRS) \
'-Wl,-z,origin' '-Wl,-rpath,$$ORIGIN/lib'
#
CFLAGS  =$(FLAGS)
CFLAGS +=-std=c99 -O2

# -- Windows -- #
ifeq ($(OS),Windows_NT)
CC 			=x86_64-w64-mingw32-gcc
FLAGS 	=-g -lm -static -static-libgcc -static-libstdc++ -lstdc++ \
-Llib/win -lopengl32 -lSDL2 -lopenal32 -I. $(IDIRS)
#
CFLAGS  =$(FLAGS)
CFLAGS +=-std=c99
endif

# -- MacOS -- #
UNAME = $(shell uname -s)
ifeq ($(UNAME),Darwin)
FLAGS   =-g -lstdc++ -framework OpenGl -framework Foundation \
-framework IOKit -lSDL2 -lphysfs -framework OpenAL \
-I. $(IDIRS) -Wno-unused-command-line-argument
#
CFLAGS  =$(FLAGS)
CFLAGS +=-std=c99 -O2
endif

# -- OpenBSD -- #
UNAME = $(shell uname -s)
ifeq ($(UNAME),OpenBSD)
CC =egcc
FLAGS += -I/usr/X11R6/include -L/usr/X11R6/lib
FLAGS:=$(filter-out -ldl,$(FLAGS)) #-ldl functionality provided in OpenBSD libc
endif

# -- Files -- #
# engine deps
# input
_EDEPS_IN = input.h
# math
_EDEPS_MA = math.h collision.h mathlib.h octree.h
# render
_EDEPS_RE = render.h scene.h camera.h defaults.h dirlight.h \
gbuffer.h mesh.h model.h msdf.h pointlight.h reflectionprobe.h \
shader.h skybox.h spotlight.h ssao.h stb_image.h stb_truetype.h \
text.h texture.h window.h framebuffer.h
# sound
_EDEPS_SO = sound.h
# util
_EDEPS_UT = util.h cache.h entity.h exe_conf.h exe_io.h \
exe_list.h iqm.h vertices.h
EDEPS		=$(patsubst %,$(EDIR)/input/%,$(_EDEPS_IN))
EDEPS	 +=$(patsubst %,$(EDIR)/math/%,$(_EDEPS_MA))
EDEPS	 +=$(patsubst %,$(EDIR)/render/%,$(_EDEPS_RE))
EDEPS	 +=$(patsubst %,$(EDIR)/sound/%,$(_EDEPS_SO))
EDEPS	 +=$(patsubst %,$(EDIR)/util/%,$(_EDEPS_UT))

# engine srcs
ENGINEOBJ = window.o glad.o mesh.o camera.o texture.o iqm.o \
framebuffer.o pointlight.o scene.o model.o dirlight.o skybox.o \
collision.o entity.o octree.o  gbuffer.o spotlight.o ssao.o engine.o \
reflectionprobe.o shader.o defaults.o input.o sound.o cache.o \
text.o msdf.o

# lib deps
_PHYSFS_DEPS =physfs_casefolding.h  physfs.h  physfs_internal.h \
physfs_lzmasdk.h  physfs_miniz.h  physfs_platforms.h
PHYSFS_DEPS  =$(patsubst %,$(LDIR)/physfs/%,$(_PHYSFS_DEPS))

# lib srcs
LOBJ		= physfs_archiver_7z.o physfs_archiver_mvl.o \
physfs_archiver_wad.o physfs_platform_posix.o \
physfs_archiver_dir.o physfs_archiver_qpak.o \
physfs_archiver_zip.o physfs_platform_qnx.o \
physfs_archiver_grp.o physfs_archiver_slb.o \
physfs_byteorder.o physfs_platform_unix.o \
physfs_archiver_hog.o physfs_archiver_unpacked.o physfs.o \
physfs_platform_windows.o physfs_archiver_iso9660.o \
physfs_archiver_vdf.o physfs_platform_os2.o physfs_unicode.o

ifeq ($(UNAME),Darwin)
LOBJ += physfs_platform_apple.o
endif

# usr srcs (add yours here)
_OBJ		=main.o game.o
_OBJ   +=$(LOBJ)
_OBJ   +=$(ENGINEOBJ)
OBJ 		=$(patsubst %,$(ODIR)/%,$(_OBJ))

# user deps
_DEPS		=game.h
DEPS		=$(patsubst %,$(IDIR)/%,$(_DEPS))

all: files game

# user src
$(ODIR)/%.o: %.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

# engine srcs
$(ODIR)/%.o: $(EDIR)/%.c $(EDEPS)
	$(CC) -c -o $@ $< $(CFLAGS)
$(ODIR)/%.o: $(EDIR)/*/%.c $(EDEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

# physfs src
$(ODIR)/%.o: $(LDIR)/physfs/%.c $(PHYSFS_DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

ifeq ($(UNAME),Darwin)
$(ODIR)/%.o: $(LDIR)/physfs/%.m $(PHYSFS_DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)
endif

game: $(OBJ)
	$(CC) -o $(BDIR)/$@ $^ $(CFLAGS)
	@echo "**success**"

files:
	mkdir -p $(ODIR)
	mkdir -p $(BDIR)/licence
	cp licence/* $(BDIR)/licence/
	(cd $(EDATA); zip -ur ../build/data.ex $(DATA) || true)
	(zip -ur build/data.ex $(DATA) || true)
ifeq ($(OS),Windows_NT)
	cp lib/win/dll/* build
endif

release-linux:
	mkdir -p $(BDIR)/lib
	rm -f $(BDIR)/lib/*
	cp $$(ldd $(BDIR)/game | grep -i libopenal | awk 'NF == 4 {print $$3}; NF == 2 {print $$1}') $(BDIR)/lib
	cp $$(ldd $(BDIR)/game | grep -i libsdl2 | awk 'NF == 4 {print $$3}; NF == 2 {print $$1}') $(BDIR)/lib

#release:
#ifeq ($(OS),Windows_NT)
#	copy /b $(BDIR)/game.exe+$(BDIR)/data.exe $(BDIR)/release.exe
#else
#	cat $(BDIR)/game $(BDIR)/data.ex > $(BDIR)/release
#	chmod +x $(BDIR)/release
#endif

.PHONY: clean release-linux

clean:
	rm -f $(ODIR)/*.o
