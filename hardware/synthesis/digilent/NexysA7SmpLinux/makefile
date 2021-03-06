#ROOT shall point to location of build.sbt
ROOT=../../../..
SWBASE=$(ROOT)/software/standalone
SOCSW=bootloader
SOCMEMSRC=$(SWBASE)/$(SOCSW)/build/$(SOCSW).elf

TOP=NexysA7SmpLinux
TOPV=$(ROOT)/hardware/netlist/$(TOP).v

RISCV_BIN?=/opt/riscv_xpacks/bin/riscv-none-embed-

all : build/latest.bit

repo-setup:
	git submodule update --init --recursive
	(cd $(ROOT)/..;git clone https://github.com/SpinalHDL/u-boot.git -b saxon u-boot)
	(cd $(ROOT)/..;git clone https://github.com/SpinalHDL/opensbi.git -b spinal)

spi-flash-sw:
	CROSS_COMPILE=$(RISCV_BIN) make -C $(ROOT)/../u-boot mrproper saxon_nexys_a7_defconfig
	CROSS_COMPILE=$(RISCV_BIN) make -C $(ROOT)/../u-boot -j$(nproc)
	CROSS_COMPILE=$(RISCV_BIN) make -C $(ROOT)/../opensbi PLATFORM=spinal/saxon/digilent/artyA7Smp SAXON_PATH=../SaxonSoc BSP=digilent/$(TOP) clean all

sdcard:
	make -C $(ROOT) formatsdcard
	make -C $(ROOT) linux2sdcard

$(TOPV) : $(SOCMEMSRC) $(ROOT)/hardware/scala/saxon/board/digilent/$(TOP).scala
	(cd $(ROOT); sbt "runMain saxon.board.digilent.$(TOP)")

generate: $(TOPV)

.PHONY: $(SOCMEMSRC)
$(SOCMEMSRC):
	mkdir -p build
	make RISCV_BIN=$(RISCV_BIN) -C $(SWBASE)/$(SOCSW) BSP=digilent/$(TOP)

build/latest.bit : $(TOPV)
	mkdir -p build
	./make_vivado_project
	cp build/vivado_project/fpga.runs/impl_1/$(TOP).bit build/latest.bit

build/latest.mcs : build/latest.bit
	./make_mcs_file

prog : build/latest.bit
	./write_fpga

flash : build/latest.mcs
	./write_flash

clean-sw:
	make -C $(SWBASE)/$(SOCSW) clean BSP=digilent/$(TOP)

soc-sw: clean-sw $(SOCMEMSRC)

.PHONY: clean
clean :
	rm -rf build
	mkdir build
	rm -f updatemem.jou
	rm -f updatemem.log

clean-all : clean clean-sw
	rm -f $(TOPV)
	rm -f $(TOPV)_*
