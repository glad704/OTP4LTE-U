# 
# Copyright (c) Microsoft Corporation
# All rights reserved. 
#
# Licensed under the Apache License, Version 2.0 (the ""License""); you
# may not use this file except in compliance with the License. You may
# obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# THIS CODE IS PROVIDED ON AN *AS IS* BASIS, WITHOUT WARRANTIES OR
# CONDITIONS OF ANY KIND, EITHER EXPRESS OR IMPLIED, INCLUDING WITHOUT
# LIMITATION ANY IMPLIED WARRANTIES OR CONDITIONS OF TITLE, FITNESS FOR
# A PARTICULAR PURPOSE, MERCHANTABLITY OR NON-INFRINGEMENT.
#
# See the Apache Version 2.0 License for specific language governing
# permissions and limitations under the License.
#
#

# Disable default rules for C files
.SUFFIXES:

# Uncomment the following if you want winddk to be the default compiler. 
# NB: you can override this from the command line, e.g: 
# prompt> COMPILER=gcc make
#
# COMPILER?=winddk
#COMPILER?=gcc
#
COMPILER?=vs

PP    = ../eNodeB/scripts/preprocesscompile-mac-$(COMPILER).sh
CP    = ../eNodeB/scripts/ccompile-mac-$(COMPILER)-inline.sh
CPDEB = ../eNodeB/scripts/ccompile-mac-$(COMPILER).sh


PPS   = ../Library/C/CompilerVS/preprocesscompile-$(COMPILER).sh
PPSF  = ../Library/C/CompilerVS/preprocesscompile-$(COMPILER)-inline.sh

ZIRIA_ROOT_CYGW=$(shell cygpath "$(ZIRIA_ROOT)")
PPZ  = $(ZIRIA_ROOT_CYGW)/scripts/preprocesscompile-$(COMPILER)-inline.sh
PPZDEB  = $(ZIRIA_ROOT_CYGW)/scripts/preprocesscompile-$(COMPILER)-debug.sh
DIFF  = $(ZIRIA_ROOT_CYGW)/tools/BlinkDiff

# Do not delete outfiles to allow quick recheck of failed tests
# use make clean to get rid of them
.PRECIOUS: %.outfile rx_test%.outfile tx_test%.outfile %.pout rx_test%.pout

# Targets
vpath %.c ../../csrc/mac
TGTS = $(patsubst %.zir, %.test, $(wildcard *.zir))

all: $(TGTS)


### Only Sora compilation/running supported


##############################################
### Actual PHY/MAC running on hardware
### Type: make mac.run
#%.c : %.zir
#	DEFINES="-DTEST_BED=1 $(DEFINES)" EXTRAOPTS='--vectorize --autolut $(EXTRAOPTS) --name=_$(basename $@)' $(PP) $< $(basename $@)

rx.c : rx.zir
	C_DEFINES='SORA_RF' DEFINES="-DTEST_BED=1 -DBW_20 $(DEFINES)" \
	EXTRAOPTS='--vectorize --autolut $(EXTRAOPTS) --name=_rx' $(PP) rx.zir rx
#	C_DEFINES='BLADE_RF' LIBS='bladeRF.lib' DEFINES="-DTEST_BED=1 -DBW_20 $(DEFINES)" \
#	EXTRAOPTS='--vectorize --autolut $(EXTRAOPTS) --name=_rx' $(PP) rx.zir rx

tx.c : tx.zir
	EXT_INC=$(ZIRIA_ROOT_CYGW)/code/WiFi/transmitter C_DEFINES='SORA_RF' \
	DEFINES="-DTEST_BED=1 -DBW_20 $(DEFINES)" \
	EXTRAOPTS='--vectorize --autolut $(EXTRAOPTS) --name=_tx' $(PP) tx.zir tx
#	EXT_INC=$(ZIRIA_ROOT_CYGW)/code/WiFi/transmitter C_DEFINES='BLADE_RF' \
#	LIBS='bladeRF.lib' DEFINES="-DTEST_BED=1 -DBW_20 $(DEFINES)" \
#	EXTRAOPTS='--vectorize --autolut $(EXTRAOPTS) --name=_tx' $(PP) tx.zir tx

mac.out: rx.c tx.c
# We need 64bit here because BladeRF won't link 32bit library
#	MODE='/p:Mode=LTEU /p:Radio=BladeRF' LIBS='bladeRF.lib' RUN_TYPE=64 EXTRAOPTS='$(EXTRAOPTS)' $(CP) mac.out
	MODE='/p:Mode=LTEU /p:Radio=Sora' RUN_TYPE=64 EXTRAOPTS='$(EXTRAOPTS)' $(CP) mac.out
# Use this one to compile 64-bit (without DebugPlot). CANNOT BE USED WITH TURBO_DECODER
#	EXTRAOPTS='$(EXTRAOPTS)' $(CP) mac.out


mac.run: mac.out
	./run_eNodeB.sh
#	PATH="$(PATH)":"/cygdrive/c/Program Files/bladeRF/x64" \
#	./run_eNodeB.sh




##############################################
### Test

test_tx.out : test_tx.zir
	EXT_INC=$(ZIRIA_ROOT_CYGW)/code/WiFi/transmitter C_DEFINES='SORA_RF' \
	DEFINES="-DTEST=1 -DBW_20 $(DEFINES)" \
	EXTRAOPTS='$(EXTRAOPTS) --bounds-check' $(PPS) $< $@
#	EXT_INC=$(ZIRIA_ROOT_CYGW)/code/WiFi/transmitter C_DEFINES='BLADE_RF' \
#	LIBS='bladeRF.lib' DEFINES="-DTEST=1 -DBW_20 $(DEFINES)" \
#	EXTRAOPTS='$(EXTRAOPTS) --bounds-check' $(PPS) $< $@

test_tx.run: test_tx.out
	./$< --input=file \
             --input-file-name=test_tx.infile \
             --input-file-mode=dbg \
	     --output=sdr --heap-size=1000000000 \
	     --sdr-central-frequency=2462000000 --sdr-rx-gain=20 \
	     --sdr-sample-rate=1536000000 --sdr-tx-gain=25


test_tx.outfile: test_tx.out test_tx.infile
	./$< --input=file \
             --input-file-name=test_tx.infile \
             --input-file-mode=dbg \
             --output-file-name=$@ \
             --output-file-mode=dbg

%.test: %.outfile
	$(DIFF) $(DIFFOPTS) -f $< -g $<.ground -d -v -n 0.8 -t 3000 -c 




##############################################
### Perf test

test_tx.pout: test_tx.zir
	EXT_INC=$(ZIRIA_ROOT_CYGW)/code/WiFi/transmitter \
	DEFINES="-DPERF=1 -DBW_20 $(DEFINES)" \
		EXTRAOPTS='--vectorize --autolut $(EXTRAOPTS)' \
		$(PPSF) $< $@


test_tx.perf: test_tx.pout 
	./$< --input=dummy --dummy-samples=1 --output=dummy



%.accept : %.outfile
	cp $< $<.ground






##############################################
### Cleaning
clean:	
	rm -f ./*expanded
	rm -f ./*exe ./*exec ./*out
	rm -f ./*.outfile
	rm -f ./compout ./perfout
	rm -f ./*.c

all-clean: clean
	rm -f ./*~
	rm -f ./*dump
