VERILOG = ../../netlist/TinyFpgaBxSocArduino.v tinyfpgabx_pll.v

generate :
	(cd ../../..; sbt "runMain saxon.board.tinyfpgabx.TinyFpgaBxSocArduino")

../../../../netlist/TinyFpgaBxSocArduino.v :
	(cd ../../..; sbt "runMain saxon.board.tinyfpgabx.TinyFpgaBxcArduino")

../../netlist/TinyFpgaBxSocArduino.v*.bin:

bin/toplevel.json : ${VERILOG} ../../netlist/TinyFpgaBxSocArduino.v*.bin
	mkdir -p bin
	rm -f TinyFpgaBxSocArduino.v*.bin
	cp ../../netlist/TinyFpgaBxSocArduino.v*.bin . | true
	yosys -v3 -p "synth_ice40 -top TinyFpgaBxSocArduino -json bin/toplevel.json" ${VERILOG}

bin/toplevel.asc : tinyfpgabx.pcf bin/toplevel.json
	nextpnr-ice40 --freq 18 --lp8k --package cm81 --json bin/toplevel.json --pcf tinyfpgabx.pcf --asc bin/toplevel.asc --opt-timing --placer heap

bin/toplevel.bin : bin/toplevel.asc
	icepack -s bin/toplevel.asc bin/toplevel.bin

compile : bin/toplevel.bin

time: bin/toplevel.bin
	icetime -tmd hx8k bin/toplevel.asc

prog : bin/toplevel.bin
	tinyprog -p bin/toplevel.bin

tinyfpgabx_pll.v   :
	icepll -i 16 -o 20 -f tinyfpgabx_pll.v -m -n tinyfpgabx_pll
clean :
	rm -rf bin
	rm -f TinyFpgaBxSocArduino.v*.bin
