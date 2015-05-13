# design name
design = mux_4to1_D4

# xst, par and etc. will report the number of actual threads supported
NUM_OF_THREADS = 4

#---------------------------------------------------------------------------
# Path to Verilog files 
#---------------------------------------------------------------------------
DEFAULT_VERILOG_DIR ?= ../verilog
USER_VERILOG_DIR    += $(DEFAULT_VERILOG_DIR) 
USER_VERILOG_DIR    += ../coregen

DEFAULT_VHDL_DIR    ?= ../vhdl
USER_VHDL_DIR       += $(DEFAULT_VHDL_DIR) 

VERILOG_FILES       += $(filter-out $(USER_VERILOG_SKIP_FILES), \
										   $(wildcard $(addsuffix /*.v, $(USER_VERILOG_DIR))))

VHDL_FILES          += $(filter-out $(USER_VHDL_SKIP_FILES), \
										   $(wildcard $(addsuffix /*.vhd, $(USER_VHDL_DIR))))

#---------------------------------------------------------------------------
# Xilinx ISE options
#---------------------------------------------------------------------------
XSYN_OPTIONS += -ifn $(design).xst -intstyle xflow

# -dd: dir to put intermediate .ngo files
XNGDBUILD_OPTIONS += $(patsubst %, -sd %, $(subst :, ,$(CORE_PATH))) \
                    -intstyle xflow \
                    -dd _ngo \
                    -uc $(design).ucf 

XMAP_OPTIONS += -intstyle xflow \
                -pr b \
                -detail \
                -xe n 
XMAP_OPTIONS += -ol high
XMAP_OPTIONS += -mt $(NUM_OF_THREADS)
XMAP_OPTIONS += -o $(design).ncd $(design).ngd 


XPAR_OPTIONS += -intstyle xflow
XPAR_OPTIONS += -xe n
XPAR_OPTIONS += -w $(design).ncd $(design)_routed.ncd $(design).pcf
XPAR_OPTIONS += -ol high
XPAR_OPTIONS += -mt $(NUM_OF_THREADS)

# 
XTRCE_OPTIONS += -intstyle xflow -v 2048 -u 32
XTRCE_OPTIONS += -xml $(design)_routed.twx
XTRCE_OPTIONS += $(design_routed.ncd $(design).pcf

XBITGEN_OPTIONS += -w -g DriveDone:Yes
XBITGEN_OPTIONS += $(design)_routed.ncd $(design).bit $(design).pcf

XCLEAN_LIST += $(design)*.prj
XCLEAN_LIST += xst *.lso *.ngr *.ngc *.xrpt *.srp
XCLEAN_LIST += *.bld *.ngd *.ngm
XCLEAN_LIST += *.bgn *.drc *.bit 
XCLEAN_LIST += *.jou *.log 
XCLEAN_LIST += *.ngo *.sdf *.nlf 
XCLEAN_LIST += _ngo xlnx* *summary.xml 
XCLEAN_LIST += *.xml *.mrp *.map *map.xrpt
XCLEAN_LIST += *.par par_*.html
XCLEAN_LIST += *.xpi *.twr *.twx
XCLEAN_LIST += *.ncd *.pcf *usage.xml
XCLEAN_LIST += *_pad.* *.pad *.ptwx *.unroutes
XCLEAN_LIST += _*  *.csv *.txt
XCLEAN_LIST += *.xwbt
XCLEAN_LIST += *.bak


#---------------------------------------------------------------------------
# Xinlinx ISE commands
#---------------------------------------------------------------------------

fpga: timing bitgen

# search for HDL files
$(design).prj : $(VERILOG_FILES) $(VHDL_FILES) 
	find . -name '*.v' -exec echo "verilog work \"{}\"" \;\
		>> $(design).prj
	for i in $(VERILOG_FILES); do\
		echo "verilog work \"$$i\"" \ >> $(design).prj; \
	done;\
	for i in $(VHDL_FILES); do\
		echo "vhdl work \"$$i\"" \ >> $(design).prj; \
	done;\

  
# xst/prj -> ngc
syn: $(design).ngc
$(design).ngc: $(design).xst $(design).prj
	-mkdir -p xst/projnav.tmp
	xst $(XSYN_OPTIONS)


CORE_PATH += ../coregen
CORE_PATH += $(XILINX_CORE_DIRS)


# merge netlists and ucf file into a design database file ngd
$(design).ngd : $(design).ngc $(design).ucf
	ngdbuild $(XNGDBUILD_OPTIONS) $(design) $(design).ngd

# ngd -> ncd and pcf
map: $(design).ncd
$(design).ncd: $(design).ngd
	map $(XMAP_OPTIONS)

# ncd -> twr (post-map STA)
$(design).twr : $(design).ncd
	trce -v32 -u32 -intstyle xflow $(design).ncd

# ncd -> routed.ncd
route: $(design)_routed.ncd
$(design)_routed.ncd: $(design).ncd
	par $(XPAR_OPTIONS)

# routed.ncd -> twr (PAR STA)
timing: $(design)_routed.twr
$(design)_routed.twr : $(design)_routed.ncd
	trce $(XTRCE_OPTIONS) 

# routed.ncd -> bit
bitgen: $(design).bit
$(design).bit: $(design)_routed.ncd 
	bitgen $(XBITGEN_OPTIONS)

# routed.ncd -> netlist after par
netlist: $(design)_routed.v
$(design)_routed.v: $(design)_routed.ncd
	rm -rf $(design)_routed.v
	netgen -ofmt verilog -sim $(design)_routed.ncd


clean:
	@rm -rf $(XCLEAN_LIST)



