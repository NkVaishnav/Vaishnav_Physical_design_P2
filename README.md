# Vaishnav_Physical_design_P2

This github repository summarizes the progress made in the Samsung PD training for the last two parts. Quick links:

- [Day-30-TCL-Programming-Workshop](https://github.com/NkVaishnav/Vaishnav_Physical_design_P2/tree/main#day-30-tcl-programming-workshop)
- [Day-31-Low-Power-Design](https://github.com/NkVaishnav/Vaishnav_Physical_design_P2/blob/main/README.md#day-31-low-power-design)


## Day 30: TCL Programming Workshop
<details>
<summary>Summary</summary>

</details>

<details>
<summary>Day 1</summary>

  On the first day , the ibjective is to craft a "tckbox" command, ensuring the TCL script seamlessly handels various user scenarios when passing a .csv file from the shell 

```
#!/bin/bash
if [ $#argv -eq 0 ]
then  
       	echo "Info: Please provide a CSV file"
	exit 1
elif [ $#argv -gt 1 ]
then 
	echo "Info: Please provide  1 CSV file"
	exit 1
else 
	if [[ $1 != *.csv && $1 != "-help" ]]
		then
			echo "Info: Please provide a .csv format file"
			exit 1
		fi
fi

if [ ! -f $1 ] || [ $1 == "-help" ]
then
	if [ $1 != "-help" ]
	then
		echo "Error: The file $1 is not found in current directory."
		exit 1
	else
		echo "USAGE: ./tclbox <csv_file>"
		echo 
		echo " where <csv file> consists of 2 columns, below keyword being in 1st column and is Case Sensitive. Please request Niharika for sample csv file."
		echo 
		echo " <Design Name> is the name of top level module."
		echo 
		echo " <Output Directory> is the name of output directory where you want to dump synthesis script, synthesized netlist and timing reports."
		echo 
		echo " <Netlist Directory> is the name of directory where all RTL netlist are present."
		echo 
		echo " <Early Library Path> is the file path of the early cell library to be used for STA."
		echo 
		echo " <Late Library Path> is file path of the late cell library to be used for STA."
		echo 
		echo " <Constraints file> is csv file path of constraints to be used for STA."
		exit 1
	fi

else
       	echo "Info: CSV file accepted"
       	tclsh tclbox.tcl $1
fi
```
We have created 6 scenarios in total using the above script which can be used 

1. No input file is being provided

2. The provided file exists but its not in the .csv  format

3. Parameters provided are more than one

4. Provided csv file doesnt exist

5. "-help" usage

6. Providing the correct file

</details>

<details>
<summary>Day 2</summary>

input file - openMSP430_design_constraints.csv


Code: tclbox.tcl

 ```
set start_time [clock clicks -microseconds]
set csv_design [lindex $argv 0]
package require csv
package require struct::matrix
struct::matrix m

set f [open $csv_design]
csv::read2matrix $f m , auto
close $f

set n_columns [m columns]
set n_rows [m rows]

puts "\nInfo:Variable values"
puts "No. of rows =  $n_rows"
puts "No. of columns = $n_columns"

m link csv_arr
set i 0
while {$i < $n_rows} {
        puts "\nInfo: Setting $csv_arr(0,$i) as '$csv_arr(1,$i)'"
        if { ![string match "*/*" $csv_arr(1,$i)] && ![string match "*.*" $csv_arr(1,$i)] } {
                        set [string map {" " "_"} $csv_arr(0,$i)] $csv_arr(1,$i)
        } else {
                set [string map {" " "_"} $csv_arr(0,$i)] [file normalize $csv_arr(1,$i)]
        }
        set i [expr {$i+1}]
}
 ```

Output of the above is as shown below 

Above shown code has set of variables defined as follows.

  - Design Name : 'openMSP430'
  -  Output Directory : './outdir_openMSP430'
  -  Netlist Directory : './verilog'
  -  Early Library Path : 'osu018_stdcells.lib'
  -  Late Library Path : 'osu018_stdcells.lib'
  -  Constraints File : 'openMSP430_design_constraints.csv'
  -  Output directory : /home/vsduser/vsdsynth/outdir_openMSP430
  -  RTL netlist directory : /home/vsduser/vsdsynth/verilog
  -  Early cell library : /home/vsduser/vsdsynth/osu018_stdcells.lib
  -  Late cell library : /home/vsduser/vsdsynth/osu018_stdcells.lib
  -  Constraints file f : /home/vsduser/vsdsynth/openMSP430_design_constraints.csv

**Check for File and Directory**

The code shown below can be used for the check of the file or directory 

```
############# FILE AND DIRECTORY EXISTENCE CHECK ###################

if { ![file isdirectory $Output_Directory] } {
        puts "\nInfo: Cannot find output directory $Output_Directory. Creating $Output_Directory"
        file mkdir $Output_Directory
} else {
        puts "\nInfo: Output directory found in path $Output_Directory"
}
# Checking if netlist directory exists if not exits
if { ![file isdirectory $Netlist_Directory] } {
        puts "\nError: Cannot find RTL netlist directory in path $Netlist_Directory. Exiting..."
        exit
} else {
        puts "\nInfo: RTL netlist directory found in path $Netlist_Directory"
}
# Checking if early cell library file exists if not exits
if { ![file exists $Early_Library_Path] } {
        puts "\nError: Cannot find early cell library in path $Early_Library_Path. Exiting..."
        exit
} else {
        puts "\nInfo: Early cell library found in path $Early_Library_Path"
}
# Checking if late cell library file exists if not exits
if { ![file exists $Late_Library_Path] } {
        puts "\nError: Cannot find late cell library in path $Late_Library_Path. Exiting..."
        exit
} else {
        puts "\nInfo: Late cell library found in path $Late_Library_Path"
}
# Checking if constraints file exists if not exits
if { ![file exists $Constraints_File] } {
        puts "\nError: Cannot find constraints file in path $Constraints_File. Exiting..."
        exit
} else {
        puts "\nInfo: Constraints file found in path $Constraints_File"
}  
```

Output of the above image is as follows

**Processing openMSP430_design_constraints.csv file**

Code used

```
# Constraints csv file data processing for convertion to format[1](excel) and SDC

puts "\nInfo: Dumping SDC constraints for $Design_Name"
::struct::matrix m1
set f1 [open $Constraints_File]
csv::read2matrix $f1 m1 , auto
close $f1
set n_rows_concsv [m1 rows]
set n_columns_concsv [m1 columns]
# Finding row number starting for CLOCKS section
set clocks_start_row [lindex [lindex [m1 search all CLOCKS] 0] 1]
# Finding column number starting for CLOCKS section
set clocks_start_column [lindex [lindex [m1 search all CLOCKS] 0] 0]
# Finding row number starting for INPUTS section
set inputs_start [lindex [lindex [m1 search all INPUTS] 0] 1]
# Finding row number starting for OUTPUTS section
set outputs_start [lindex [lindex [m1 search all OUTPUTS] 0] 1]

puts "\nInfo: Listing value of variables for user debug"
puts "Number of rows in CSV file = $n_rows_concsv"
puts "Number of columns in CSV file = $n_columns_concsv"
puts "CLOCKS starting row in CSV file = $clocks_start_row"
puts "CLOCKS starting column in CSV file = $clocks_start_column"
puts "INPUTS starting row in CSV file = $inputs_start "
puts "OUTPUTS starting row in CSV file = $outputs_start "

```
Output of the above code is as follows


</details>

<details>
<summary>Day 3</summary>
On Day 3, the focus lies in examining dclock abnd input constraints stored in a CSV file and producing an SDC file. The process involves utilizing matrix search algorithms and a specialized algorithm to differentate between bus and bit inputs. The constraints from the CSV file are processed for clocks, resulting in SDC commands saved in the '.sdc' file (clock name : dco_clk_tclbox).
Refer to the code : 'tclbox.tcl'

```
# Conversion of constraints csv file to SDC
# CLOCKS section

#puts "$n_columns_concsv"
set clk_erd_st_col [lindex [lindex [m1 search rect $clocks_start_column $clocks_start_row [expr { $n_columns_concsv - 1}] [expr {$inputs_start-1}] early_rise_delay] 0 ] 0 ]
set clk_efd_st_col [lindex [lindex [m1 search rect $clocks_start_column $clocks_start_row [expr { $n_columns_concsv - 1}] [expr {$inputs_start-1}] early_fall_delay] 0 ] 0 ]
set clk_lrd_st_col [lindex [lindex [m1 search rect $clocks_start_column $clocks_start_row [expr { $n_columns_concsv - 1}] [expr {$inputs_start-1}] late_rise_delay] 0 ] 0 ]
set clk_lfd_st_col [lindex [lindex [m1 search rect $clocks_start_column $clocks_start_row [expr { $n_columns_concsv - 1}] [expr {$inputs_start-1}] late_fall_delay] 0 ] 0 ]
# Finding column number starting for clock transition in CLOCKS section
set clk_ers_st_col [lindex [lindex [m1 search rect $clocks_start_column $clocks_start_row [expr { $n_columns_concsv - 1}] [expr {$inputs_start-1}] early_rise_slew] 0 ] 0 ]
set clk_efs_st_col [lindex [lindex [m1 search rect $clocks_start_column $clocks_start_row [expr { $n_columns_concsv - 1}] [expr {$inputs_start-1}] early_fall_slew] 0 ] 0 ]
set clk_lrs_st_col [lindex [lindex [m1 search rect $clocks_start_column $clocks_start_row [expr { $n_columns_concsv - 1}] [expr {$inputs_start-1}] late_rise_slew] 0 ] 0 ]
set clk_lfs_st_col [lindex [lindex [m1 search rect $clocks_start_column $clocks_start_row [expr { $n_columns_concsv - 1}] [expr {$inputs_start-1}] late_fall_slew] 0 ] 0 ]
# Finding column number starting for frequency and duty cycle in CLOCKS section only
set clk_freq_st_col [lindex [lindex [m1 search rect $clocks_start_column $clocks_start_row [expr { $n_columns_concsv - 1}] [expr {$inputs_start-1}] frequency] 0 ] 0 ]
set clk_dc_st_col [lindex [lindex [m1 search rect $clocks_start_column $clocks_start_row [expr { $n_columns_concsv - 1}] [expr {$inputs_start-1}] duty_cycle] 0 ] 0 ]

# Creating .sdc file with design name in output directory and opening it in write mode
set sdc_file [open $Output_Directory/$Design_Name.sdc "w"]
# Setting variables for actual clock row start and end
set i [expr {$clocks_start_row+1}]
set end_of_clocks [expr {$inputs_start-1}]

puts "\nInfo-SDC: Working on clock constraints and creating clocks. Please wait"

# while loop to write constraint commands to .sdc file
while { $i < $end_of_clocks } {
	#Create SDC command to create clocks.
	puts -nonewline $sdc_file "\ncreate_clock -name [concat [m1 get cell 0 $i]_tclbox] -period [m1 get cell $clk_freq_st_col $i] -waveform \{0 [expr {[m1 get cell $clk_freq_st_col $i]*[m1 get cell $clk_dc_st_col $i]/100}]\} \[get_ports [m1 get cell 0 $i]\]"
	# set_clock_transition SDC command to set clock transition values
	puts -nonewline $sdc_file "\nset_clock_transition -min -rise [m1 get cell $clk_ers_st_col $i] \[get_clocks [m1 get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -min -fall [m1 get cell $clk_efs_st_col $i] \[get_clocks [m1 get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -max -rise [m1 get cell $clk_lrs_st_col $i] \[get_clocks [m1 get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -max -fall [m1 get cell $clk_lfs_st_col $i] \[get_clocks [m1 get cell 0 $i]\]"
	# set_clock_latency SDC command to set clock latency values
	puts -nonewline $sdc_file "\nset_clock_latency -source -early -rise [m1 get cell $clk_erd_st_col $i] \[get_clocks [m1 get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -early -fall [m1 get cell $clk_efd_st_col $i] \[get_clocks [m1 get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -late -rise [m1 get cell $clk_lrd_st_col $i] \[get_clocks [m1 get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -late -fall [m1 get cell $clk_lfd_st_col $i] \[get_clocks [m1 get cell 0 $i]\]"
	set i [expr {$i+1}]
}
set clocks_start_row_actual [expr {$clocks_start_row+1}]
puts "\n Clocks created in .sdc file. Values for debugging: "
puts "\n Clock early rise delay start column in constraint file = $clk_erd_st_col"
puts "\n Clock early fall delay start column in constraint file = $clk_efd_st_col"
puts "\n Clock late rise delay start column in constraint file = $clk_lrd_st_col"
puts "\n Clock late fall delay start column in constraint file = $clk_lfd_st_col"
puts "\n Clock early rise slew start column in constraint file = $clk_ers_st_col"
puts "\n Clock early fall slew start column in constraint file = $clk_efs_st_col"
puts "\n Clock late rise slew start column in constraint file = $clk_lrs_st_col"
puts "\n Clock late fall slew start column in constraint file = $clk_lfs_st_col"
puts "\n Clock frequency start column in constraint file = $clk_freq_st_col"
puts "\n Clock duty cycle start column in constraint file = $clk_dc_st_col"
puts "\n Clock actual starting row = $clocks_start_row_actual"
puts "\n Clock actual ending row = $end_of_clocks"
```
Output of the above code is as shown below 


</details>

<details>
<summary>Day 4</summary>
On Day 4, the agenda involves scripting for the Yosys synthesis, specifically tackling tasks like handling the output section, generating SDC, inspecting Yosys hierarchy and resolving errors. The script processes the constraints CSV for the OUTPUTS, depositing relavant SDC commands into the '.sdc' 
Check out the code: 'tclbox.tcl'

```
# Finding column number starting for output clock latency in OUTPUTS section only
set op_erd_st_col [lindex [lindex [m1 search rect $clocks_start_column $outputs_start [expr {$n_columns_concsv-1}] [expr {$n_rows_concsv-1}] early_rise_delay] 0 ] 0 ]
set op_efd_st_col [lindex [lindex [m1 search rect $clocks_start_column $outputs_start [expr {$n_columns_concsv-1}] [expr {$n_rows_concsv-1}] early_fall_delay] 0 ] 0 ]
set op_lrd_st_col [lindex [lindex [m1 search rect $clocks_start_column $outputs_start [expr {$n_columns_concsv-1}] [expr {$n_rows_concsv-1}] late_rise_delay] 0 ] 0 ]
set op_lfd_st_col [lindex [lindex [m1 search rect $clocks_start_column $outputs_start [expr {$n_columns_concsv-1}] [expr {$n_rows_concsv-1}] late_fall_delay] 0 ] 0 ]

# Finding column number starting for output related clock in OUTPUTS section only
set op_rc_st_col [lindex [lindex [m1 search rect $clocks_start_column $outputs_start [expr {$n_columns_concsv-1}] [expr {$n_rows_concsv-1}] clocks] 0 ] 0 ]

# Finding column number starting for output load in OUTPUTS section only
set op_load_st_col [lindex [lindex [m1 search rect $clocks_start_column $outputs_start [expr {$n_columns_concsv-1}] [expr {$n_rows_concsv-1}] load] 0 ] 0 ]

# Setting variables for actual input row start and end
set i [expr {$outputs_start+1}]
set end_of_outputs [expr {$n_rows_concsv-1}]

puts "\nInfo-SDC: Working on output constraints.."
puts "\nInfo-SDC: Categorizing output ports as bits and busses"

# while loop to write constraint commands to .sdc file
while { $i < $end_of_outputs } {
	# Checking if input is bussed or not
	set netlist [glob -dir $Netlist_Directory *.v]
	set tmp_file [open /tmp/1 w]
	foreach f $netlist {
		set fd [open $f]
		while { [gets $fd line] != -1 } {
			set pattern1 " [m1 get cell 0 $i];"
			if { [regexp -all -- $pattern1 $line] } {
				set pattern2 [lindex [split $line ";"] 0]
				if { [regexp -all {output} [lindex [split $pattern2 "\S+"] 0]] } {
					set s1 "[lindex [split $pattern2 "\S+"] 0] [lindex [split $pattern2 "\S+"] 1] [lindex [split $pattern2 "\S+"] 2]"
					puts -nonewline $tmp_file "\n[regsub -all {\s+} $s1 " "]"
				}
			}
		}
	close $fd
	}
	close $tmp_file
	set tmp_file [open /tmp/1 r]
	set tmp2_file [open /tmp/2 w]
	puts -nonewline $tmp2_file "[join [lsort -unique [split [read $tmp_file] \n]] \n]"
	close $tmp_file
	close $tmp2_file
	set tmp2_file [open /tmp/2 r]
	set count [llength [read $tmp2_file]]
	close $tmp2_file
	if {$count > 2} {
		set op_ports [concat [m1 get cell 0 $i]*]
	} else {
		set op_ports [m1 get cell 0 $i]
	}

	# set_output_delay SDC command to set output latency values
	puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [m1 get cell $op_rc_st_col $i]\] -min -rise -source_latency_included [m1 get cell $op_erd_st_col $i] \[get_ports $op_ports\]"
	puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [m1 get cell $op_rc_st_col $i]\] -min -fall -source_latency_included [m1 get cell $op_efd_st_col $i] \[get_ports $op_ports\]"
	puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [m1 get cell $op_rc_st_col $i]\] -max -rise -source_latency_included [m1 get cell $op_lrd_st_col $i] \[get_ports $op_ports\]"
	puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [m1 get cell $op_rc_st_col $i]\] -max -fall -source_latency_included [m1 get cell $op_lfd_st_col $i] \[get_ports $op_ports\]"

	# set_load SDC command to set load values
	puts -nonewline $sdc_file "\nset_load [m1 get cell $op_load_st_col $i] \[get_ports $op_ports\]"

	set i [expr {$i+1}]
}
close $sdc_file
puts "\nInfo-SDC: SDC created. Please use constraints in path $Output_Directory/$Design_Name.sdc"
```
Output of the above script is as shown below 

**Script for the Hierarchy check**

Check out the code: 'tclbox.tcl'

```
# Hierarchy Check
puts "\nInfo: Creating hierarchy check script to be used by Yosys"
set data "read_liberty -lib -ignore_miss_dir -setattr blackbox ${Late_Library_Path}"
set filename "$Design_Name.hier.ys"
set fileId [open $Output_Directory/$filename "w"]
puts -nonewline $fileId $data
set netlist [glob -dir $Netlist_Directory *.v]
foreach f $netlist {
	set data $f
	puts -nonewline $fileId "\nread_verilog $f"
	puts "\nInfo: Netlist being read for user debug: $f" 
}
puts -nonewline $fileId "\nhierarchy -check"
close $fileId

# Hierarchy check error handling
# Hierarchy check error handling done to see any errors popping up in above script.
# Running hierarchy check in yosys by dumping log to log file and catching execution message
set error_flag [catch {exec yosys -s $Output_Directory/$Design_Name.hier.ys >& $Output_Directory/$Design_Name.hierarchy_check.log} msg]
puts "Errfor flag value for user debug: $error_flag"
if { $error_flag } {
	set filename "$Output_Directory/$Design_Name.hierarchy_check.log"
	# EDA tool specific hierarchy error search pattern
	set pattern {referenced in module}
	set count 0
	set fid [open $filename r]
	while { [gets $fid line] != -1 } {
		incr count [regexp -all -- $pattern $line]
		if { [regexp -all -- $pattern $line] } {
			puts "\nError: Module [lindex $line 2] is not part of design $Design_Name. Please correct RTL in the path '$Netlist_Directory'"
			puts "\nInfo: Hierarchy check FAIL"
		}
	}
	close $fid
	puts "\nInfo: Please find hierarchy check details in '[file normalize $Output_Directory/$Design_Name.hierarchy_check.log]' for more info. Exiting..."
	
} else {
	puts "\nInfo: Hierarchy check PASS"
	puts "\nInfo: Please find hierarchy check details in '[file normalize $Output_Directory/$Design_Name.hierarchy_check.log]' for more info"
}
```

Output of the above script is as shown below


  
</details>

<details>
<summary>Day 5</summary>
</details>

<details>
<summary>Day 5 P2</summary>
</details>
## Day 31: Low Power Design 

<details>
<summary>Summary</summary>


</details>
