JULIAHOME = $(abspath ../..)
include $(JULIAHOME)/Make.inc

override CFLAGS += $(JCFLAGS)
override CXXFLAGS += $(JCXXFLAGS)

NAME = flisp
EXENAME = $(NAME)
LIBTARGET = lib$(NAME)

SRCS = flisp.c builtins.c string.c equalhash.c table.c iostream.c \
        julia_extensions.c

OBJS = $(SRCS:%.c=%.o)
DOBJS = $(SRCS:%.c=%.do)
LLTDIR = ../support
LLT = $(LLTDIR)/libsupport.a $(LIBUV)

FLAGS = -Wall -Wno-strict-aliasing -I$(LLTDIR) $(CFLAGS) \
	-DUSE_COMPUTED_GOTO $(HFILEDIRS:%=-I%) -I$(LIBUV_INC) $(LIBDIRS:%=-L%) \
        -fvisibility=hidden -DLIBRARY_EXPORTS
LIBFILES = $(LLT)
LIBS = $(LIBFILES)
ifneq ($(OS),WINNT)
LIBS += -lpthread
endif

DEBUGFLAGS += $(FLAGS)
SHIPFLAGS += $(FLAGS)

default: release

release: $(EXENAME)

debug: $(EXENAME)-debug

HEADERS = $(wildcard *.h) $(LIBUV_INC)/uv.h

%.o: %.c $(HEADERS)
	@$(call PRINT_CC, $(CC) $(CPPFLAGS) $(SHIPFLAGS) -DNDEBUG -c $< -o $@)
%.do: %.c $(HEADERS)
	@$(call PRINT_CC, $(CC) $(CPPFLAGS) $(DEBUGFLAGS) -c $< -o $@)

flisp.o:   flisp.c cvalues.c types.c flisp.h print.c read.c equal.c
flisp.do:  flisp.c cvalues.c types.c flisp.h print.c read.c equal.c
flmain.o:  flmain.c flisp.h
flmain.do: flmain.c flisp.h

$(LLT): $(LLTDIR)/*.h $(LLTDIR)/*.c
	cd $(LLTDIR) && $(MAKE)

$(LIBTARGET)-debug.a: $(DOBJS)
	rm -rf $@
	@$(call PRINT_LINK, $(AR) -rcs $@ $(DOBJS))
	ln -sf $@ $(LIBTARGET).a

$(LIBTARGET).a: $(OBJS)
	rm -rf $@
	@$(call PRINT_LINK, $(AR) -rcs $@ $(OBJS))

$(EXENAME)-debug: $(DOBJS) $(LIBFILES) $(LIBTARGET)-debug.a flmain.do
	@$(call PRINT_CC, $(CC) $(DEBUGFLAGS) $(DOBJS) flmain.do -o $(EXENAME)-debug $(LIBTARGET).a $(LIBS) $(OSLIBS))
	$(call spawn,./$(EXENAME)-debug) unittest.lsp

$(EXENAME): $(OBJS) $(LIBFILES) $(LIBTARGET).a flmain.o
	@$(call PRINT_CC, $(CC) $(SHIPFLAGS) $(OBJS) flmain.o $(LDFLAGS) -o $(EXENAME) $(LIBTARGET).a $(LIBS) $(OSLIBS))
	$(call spawn,./$(EXENAME)) unittest.lsp

clean:
	rm -f *.o
	rm -f *.do
	rm -f *.a
	rm -f $(EXENAME)
	rm -f $(EXENAME)-debug
