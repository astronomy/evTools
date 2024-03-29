# Compile the program and put the executable in ~/usr/bin

.PHONY: clean cleanwarnings code_version exit
.PRECIOUS: %.o


# No text/space after statements below!
FORT = gfortran#  Use GNU fortran compiler >=4.4
# FORT = ifort#     Use Intel fortran compiler
# FORT = nagfor#

WARN = yes#    # compile with compile-time warnings
CHECK = yes#   # compile with run-time checks
DEBUG = yes#   # compile with debug options
F2003 = yes#   # compile with Fortran 2003 standard


# General:
OPT = -O0
# OPT = -O1
# OPT = -O2
# OPT = -O3


# Directories for the source, object, module, libary and executable files:
SRCDIR = src
OBJDIR = obj
MODDIR = mod

BINDIR  = $(HOME)/usr/bin
INCLDIR = $(HOME)/usr/include
LIBDIR  = $(HOME)/usr/lib

STDOBJ = $(OBJDIR)/functions.o $(OBJDIR)/code_version.o
PLTOBJ = $(OBJDIR)/plt_functions.o
MDLOBJ = $(OBJDIR)/mdl_functions.o
PLOTOBJ = $(OBJDIR)/plotfunctions.o 
MODOBJ = 


# PGPLOT flags:
PGPLOT_LFLAGS = -lpgplot



# ifort:
ifeq ($(FORT),ifort)
   CFLAGS += #-I$(INCLDIR)/libSUFR/ifort
   LFLAGS += #-L$(LIBDIR) -lSUFR_ifort
   STATLIBS += #$(LIBDIR)/libSUFR_ifort.a

   # To develop code  (don't mix -check bounds with -O>0!):
   ifeq ($(CHECK),yes)
      OPT = -O0
      CFLAGS += -check all -check noarg_temp_created
   endif

   ifeq ($(WARN),yes)
      CFLAGS += -warn all -stand f03
   endif

   ifeq ($(F2003),yes)
      CFLAGS += -stand f03 -diag-disable 6894
   endif

   CFLAGS += -vec-report0 -traceback
   CFLAGS += -module $(MODDIR) -I$(MODDIR)

   # To debug code:
   ifeq ($(DEBUG),yes)
      OPT = -O0
      CFLAGS += -g
   endif


   CFLAGS += -nogen-interfaces
   STATIC = -static

   # OPT += -ipo
   # LFLAGS += -ipo
   # OPT += -xsse4.1
   # OPT += -xhost
   # CFLAGS += -static $(STATLIBS)
   # CFLAGS += -prof-gen -prof-dir./  #Don't use -ipo. Run the code thus compiled (a few times) and use -prof-use below for the final version
   # CFLAGS += -prof-use -prof-dir./

endif


# gfortran: similar in speed to ifort, but file reading is ~2.5x slower!
# ifneq (,$(filter $(FORT),gfortran gfortran-4.5))
ifeq ($(FORT),gfortran)
   CFLAGS += -I$(INCLDIR)/libSUFR/gfortran
   LFLAGS += -L$(LIBDIR) -lSUFR_gfortran
   STATLIBS += # $(LIBDIR)/libSUFR_gfortran.a

   CFLAGS += -fwhole-file

   # Overrule PGPLOT with PG2PLplot:
   CFLAGS += -I ~/usr/include/plplot_MvdS/${FORT} -I/usr/lib64/fortran/modules/plplot/
   PLPLOT_LFLAGS = -lplplotfortran -lpg2plplot_gfortran  # -lpg2plplot  #  -lplplot_MvdS_gfortran

   # Run-time checks:
   ifeq ($(CHECK),yes)
      OPT = -O0
      # CFLAGS += -fbounds-check
      CFLAGS += -fcheck=all
      # CFLAGS += -fcheck-array-temporaries  #Doesn't seem to work...
      CFLAGS += -fbacktrace
   endif

   # Warnings:
   ifeq ($(WARN),yes)
      CFLAGS += -Wall -Wextra
      CFLAGS += -Werror
      CFLAGS += -Wcharacter-truncation -Wunderflow
      CFLAGS += -fmax-errors=10
      ifeq ($(FORT),gfortran-4.5.0)
         CFLAGS += -fwhole-file
         # CFLAGS += -fwhole-program
      endif
   endif

   # F2003:
   ifeq ($(F2003),yes)
      CFLAGS += -std=f2008 -fall-intrinsics -pedantic
   else
      CFLAGS += -ffree-line-length-none
   endif

   # Debug code:
   ifeq ($(DEBUG),yes)
      OPT = -O0
      CFLAGS += -g
   endif
   CFLAGS += -I$(MODDIR) -J$(MODDIR)
   STATIC = -static
endif


# nagfor:
ifeq ($(FORT),nagfor)
   ifeq ($(CHECK),yes)
      CFLAGS = -O0
      CFLAGS += -C=all
   endif
   ifneq ($(WARN),yes)
      CFLAGS += -w=all  # Warnings are on by default
   endif
   CFLAGS += -colour -f2003 -u -gline
   STDOBJ += $(OBJDIR)/nagfor.o
   CFLAGS += -I$(MODDIR) -mdir $(MODDIR)
   STATIC = -Bstatic
endif





all: tools_all plot_all

plot_all: plotmdl plotmdln plotmod plotplt
tools_all: dat2plt findplt getgrid grid listmdl listmod listplt makerun mergeplt plt2ce plt2dat plt2obs selplt

$(OBJDIR)/%.o: $(SRCDIR)/%.f90
	$(FORT) $(OPT) $(CFLAGS) -c $< -o $@

exit:


# Plotting with PGPLOT or PLplot:

plotmdl: $(STDOBJ) $(PLOTOBJ) $(MDLOBJ) $(OBJDIR)/plotmdl.o
	$(FORT) $(LFLAGS) $(PGPLOT_LFLAGS) -o $(BINDIR)/plotmdl $(STDOBJ) $(PLOTOBJ) $(MDLOBJ) $(OBJDIR)/plotmdl.o

plotmdln: $(STDOBJ) $(PLOTOBJ) $(MDLOBJ) $(OBJDIR)/plotmdln.o
	$(FORT) $(LFLAGS) $(PGPLOT_LFLAGS) -o $(BINDIR)/plotmdln $(STDOBJ) $(PLOTOBJ) $(MDLOBJ) $(OBJDIR)/plotmdln.o


plotmod: $(STDOBJ) $(MODOBJ) $(OBJDIR)/plotmod.o
	$(FORT) $(LFLAGS) $(PGPLOT_LFLAGS) -o $(BINDIR)/plotmod $(STDOBJ) $(MODOBJ) $(OBJDIR)/plotmod.o

plotplt: $(STDOBJ) $(PLOTOBJ) $(PLTOBJ) $(OBJDIR)/plotplt.o
# $(FORT) $(LFLAGS) $(PGPLOT_LFLAGS) -o $(BINDIR)/plotplt $(STDOBJ) $(PLOTOBJ) $(PLTOBJ) $(OBJDIR)/plotplt.o
	$(FORT) $(LFLAGS) $(PLPLOT_LFLAGS) -o $(BINDIR)/plotplt $(STDOBJ) $(PLOTOBJ) $(PLTOBJ) $(OBJDIR)/plotplt.o



# Tools:

dat2plt: $(STDOBJ) $(OBJDIR)/dat2plt.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/dat2plt $(STDOBJ) $(OBJDIR)/dat2plt.o

convert_initdat: $(STDOBJ) $(OBJDIR)/convert_initdat.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/convert_initdat $(STDOBJ) $(OBJDIR)/convert_initdat.o

findplt: $(STDOBJ) $(OBJDIR)/findplt.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/findplt $(STDOBJ) $(OBJDIR)/findplt.o

getgrid: $(STDOBJ) $(OBJDIR)/getgrid.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/getgrid $(STDOBJ) $(OBJDIR)/getgrid.o

grid: $(STDOBJ) $(OBJDIR)/grid.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/grid $(STDOBJ) $(OBJDIR)/grid.o

listmdl: $(STDOBJ) $(MDLOBJ) $(OBJDIR)/listmdl.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/listmdl $(STDOBJ) $(MDLOBJ) $(OBJDIR)/listmdl.o

listmod: $(STDOBJ) $(OBJDIR)/listmod.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/listmod $(STDOBJ) $(OBJDIR)/listmod.o

listplt: $(STDOBJ) $(OBJDIR)/listplt.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/listplt $(STDOBJ) $(OBJDIR)/listplt.o

makerun: $(STDOBJ) $(OBJDIR)/init_functions.o $(OBJDIR)/makerun.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/makerun $(STDOBJ) $(OBJDIR)/init_functions.o $(OBJDIR)/makerun.o

mergeplt: $(STDOBJ) $(OBJDIR)/mergeplt.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/mergeplt $(STDOBJ) $(OBJDIR)/mergeplt.o



plt2ce: $(STDOBJ) $(OBJDIR)/plt2ce.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/plt2ce $(STDOBJ) $(OBJDIR)/plt2ce.o

plt2dat: $(STDOBJ) $(PLTOBJ) $(OBJDIR)/plt2dat.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/plt2dat $(STDOBJ) $(PLTOBJ) $(OBJDIR)/plt2dat.o

plt2obs: $(STDOBJ) $(OBJDIR)/plt2obs.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/plt2obs $(STDOBJ) $(OBJDIR)/plt2obs.o

selplt: $(STDOBJ) $(PLTOBJ) $(OBJDIR)/selplt.o
	$(FORT) $(LFLAGS) -o $(BINDIR)/selplt $(STDOBJ) $(PLTOBJ) $(OBJDIR)/selplt.o




# General:

$(SRCDIR)/code_version.f90: code_version    #code_version is PHONY and always out-of-date
	@echo 'Generating '$(SRCDIR)'/code_version.f90'
	@echo '!> \file code_version.f90  Source file automatically generated by the evTools Makefile' > $(SRCDIR)/code_version.f90
	@echo '!!                         to report the code version and compiler settings used.' >> $(SRCDIR)/code_version.f90
	@echo 'subroutine print_code_version(unit)' >> $(SRCDIR)/code_version.f90
	@echo '   implicit none' >> $(SRCDIR)/code_version.f90
	@echo '   integer, intent(in) :: unit' >> $(SRCDIR)/code_version.f90
	@echo "   character :: svn_revision*(99) = '"`svnversion`"'" >> $(SRCDIR)/code_version.f90
	@echo "   character :: compile_date*(99) = '"`date`"'" >> $(SRCDIR)/code_version.f90
	@echo "   character :: compiler*(99) = '$(FORT)'" >> $(SRCDIR)/code_version.f90
	@echo "   character :: compiler_flags*(99) = '$(OPT)'" >> $(SRCDIR)/code_version.f90
	@echo "   character :: program_name*(99)" >> $(SRCDIR)/code_version.f90
	@echo "   call get_command_argument(0,program_name)" >> $(SRCDIR)/code_version.f90
	@echo "   write(unit,'(A)')' '//trim(program_name)//', svn revision '//trim(svn_revision)// &" >> $(SRCDIR)/code_version.f90
	@echo "        ', compiled on '//trim(compile_date)//' with '//trim(compiler)//' '//trim(compiler_flags)" >> $(SRCDIR)/code_version.f90
	@echo 'end subroutine print_code_version' >> $(SRCDIR)/code_version.f90
	@touch -d "1 Jan 2001" $(SRCDIR)/code_version.f90  #Make it old, so that this file is not a reason to relink - this only works if binaries are in current dir, i.e. binary name == target name.  Now, you just don't recompile this source file. Even then, if you don't recompile/relink, you won't get the correct compiler/date
	@echo -e "Using Fortran compiler "`which $(FORT)`


clean:
	rm -f $(OBJDIR)/*.o $(MODDIR)/*.mod $(SRCDIR)/code_version.f90

cleanwarnings:
	rm -f *__genmod.f90 *__genmod.mod

