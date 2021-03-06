# ======================================================================
# NOTE: make all
# -----
#
# ======================================================================
.PHONY: prep clean vars

ROOTCINT      = $(ROOTSYS)/bin/rootcint
ROOTCFLAGS    = $(shell $(ROOTSYS)/bin/root-config --cflags)
ROOTGLIBS     = $(shell $(ROOTSYS)/bin/root-config --glibs)

ifdef SCRAM_ARCH
  CXX         := $(shell scramv1 tool info gcc-cxxcompiler | grep CXX= | sed s/CXX=//)
else 
  ifdef CXXCOMPILER
    CXX         := $(CXXCOMPILER)
  else 
    CXX         := c++
  endif
endif

CXXFLAGS      = -g -O0 -Wall -fPIC -pipe
#CXXFLAGS      = -g -O3 -Wall -fPIC -pipe
LD            = $(CXX)
LDFLAGS       = -g
SOFLAGS       = -shared

CXXFLAGS     += $(ROOTCFLAGS)
GLIBS         = $(filter-out -lz, $(ROOTGLIBS)) -lTMVA
EXTHEADERS    = -I..


# -- Files
CSTUFF = preselection.o 
READER = treeReader01.o bmmReader.o \
         AnalysisCuts.o AnalysisDistribution.o initFunc.o \
         candAna.o candAnaMuMu.o candAnaBu2JpsiK.o candAnaBs2JpsiPhi.o candAnaDstar.o candAnaHh.o candAnaBd2DstarPi.o
DICT = ${READER:.o=Dict.o}
DICTHEADERS = ${READER:.o=Dict.h}
DICTSOURCES = ${READER:.o=Dict.cc}


# -- Default rules
$(addprefix obj/,%.o) : %.cc %.hh %.icc
	$(CXX) $(CXXFLAGS) $(EXTHEADERS) -c $< -o $@

$(addprefix obj/,%.o) : %.cc %.hh
	$(CXX) $(CXXFLAGS) $(EXTHEADERS) -c $< -o $@

$(addprefix obj/,%.o) : %.cc 
	$(CXX) $(CXXFLAGS) $(EXTHEADERS) -c $< -o $@

%Dict.cc : %.hh %LinkDef.h
	$(ROOTCINT) -f $@ -c $(EXTHEADERS) $^ 

%Dict.cc : %.hh
	$(ROOTCINT) -f $@ -c $(EXTHEADERS) $< 


# -- Targets
all: vars prep lib bin

lib: $(addprefix obj/,$(CSTUFF) $(READER)  $(DICT))
	$(CXX) $(SOFLAGS) $(addprefix obj/,$(CSTUFF) $(READER) $(DICT)) -o lib/libAnaClasses.so $(GLIBS) -lMinuit lib/libAna00.so lib/libUtil.so


bin: lib obj/runBmm.o
	$(LD) $(LDFLAGS) -o bin/runBmm $(GLIBS) obj/runBmm.o lib/libAnaClasses.so lib/libAna00.so lib/libUtil.so


# -- preparatory setup
prep:
	mkdir -p obj bin lib
	cd lib && ln -f -s ../../rootio/lib/libAna00.so && cd - 
	cd lib && ln -f -s ../../rootio/lib/libUtil.so && cd - 

# -- clean up
clean:
	rm -f $(addprefix obj/,$(READER) $(DICT) runBmm.o) 
	rm -f $(DICTHEADERS)  $(DICTSOURCES) 
	rm -f bin/*
	rm -f lib/*

# -- ensure that the environment variable DELPHES is set
vars:
ifndef CMSSW_BASE
    $(error CMSSW_BASE is undefined, please run cmsenv somewhere)
endif
