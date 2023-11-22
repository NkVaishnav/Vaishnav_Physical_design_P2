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

**Variable Creation**


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


</details>

<details>
<summary>Day 3</summary>
On Day 3, the focus lies in examining dclock abnd input constraints stored in a CSV file and producing an SDC file. The process involves utilizing matrix search algorithms and a specialized algorithm to differentate between bus and bit inputs. The constraints from the CSV file are processed for clocks, resulting in SDC commands saved in the '.sdc' file (clock name : dco_clk_tcl).
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

![image](https://github.com/NkVaishnav/Vaishnav_Physical_design_P2/assets/142480622/3d9ddaa4-8190-462e-8447-b0aa8df191e7)


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

</details>

<details>
<summary>Day 5</summary>

 **Advanced Scripting techniques**


```
# Main Synthesis Script for yosys
# ---------------------
puts "\nInfo: Creating main synthesis script to be used by Yosys"
set data "read_liberty -lib -ignore_miss_dir -setattr blackbox ${Late_Library_Path}"
set filename "$Design_Name.ys"
set fileId [open $Output_Directory/$filename "w"]
puts -nonewline $fileId $data
set netlist [glob -dir $Netlist_Directory *.v]
foreach f $netlist {
	puts -nonewline $fileId "\nread_verilog $f"
}
puts -nonewline $fileId "\nhierarchy -top $Design_Name"
puts -nonewline $fileId "\nsynth -top $Design_Name"
puts -nonewline $fileId "\nsplitnets -ports -format ___\ndfflibmap -liberty ${Late_Library_Path} \nopt"
puts -nonewline $fileId "\nabc -liberty ${Late_Library_Path}"
puts -nonewline $fileId "\nflatten"
puts -nonewline $fileId "\nclean -purge\niopadmap -outpad BUFX2 A:Y -bits\nopt\nclean"
puts -nonewline $fileId "\nwrite_verilog $Output_Directory/$Design_Name.synth.v"
close $fileId
puts "\nInfo: Synthesis script created and can be accessed from path $Output_Directory/$Design_Name.ys"

```

Below script is ran for synthesis and handling errors

```
puts "\nInfo: Running synthesis......."
# Main synthesis error handling
# Running main synthesis in yosys by dumping logs to the log directory and catching execution message
if { [catch {exec yosys -s $Output_Directory/$Design_Name.ys >& $Output_Directory/$Design_Name.synthesis.log} msg] } {
	puts "\nError: Synthesis failed due to errors. Please refer to log $Output_Directory/$Design_Name.synthesis.log for errors. Exiting...."
	exit
} else {
	puts "\nInfo: Synthesis finished successfully"
}
puts "\nInfo: Please refer to log $Output_Directory/$Design_Name.synthesis.log"
```

 Procs can be used to create user-defined commands.
 Different procs used throught the training is given below:

1. reopenStdout.proc
```
  #!/bin/tclsh
#proc to redirect screen log to file
proc reopenStdout {file} {
    close stdout
    open $file w       
}
```
2. set_multi_cpu_usage.proc
```
#!/bin/tclsh
proc set_multi_cpu_usage {args} {
        array set options {-localCpu <num_of_threads> -help "" }
        foreach {switch value} [array get options] {
        puts "Option $switch is $value"
        }
        while {[llength $args]} {
        puts "llength is [llength $args]"
        puts "lindex 0 of \"$args\" is [lindex $args 0]"
                switch -glob -- [lindex $args 0] {
                -localCpu {
                           puts "old args is $args"
                           set args [lassign $args - options(-localCpu)]
                           puts "new args is \"$args\""
                           puts "set_num_threads $options(-localCpu)"
                          }
                -help {
                           puts "old args is $args"
                           set args [lassign $args - options(-help) ]
                           puts "new args is \"$args\""
                           puts "Usage: set_multi_cpu_usage -localCpu <num_of_threads> -help"
                           puts "\t-localCpu - To limit number of threads used"
                           puts "\t-help - To print details of proc"
                      }
                }
        }
}
#set_multi_cpu_usage -localCpu 5 -help
```
3. read_lib.proc

```
#!/bin/tclsh
proc read_lib args {
	# Setting command parameter options and its values
	array set options {-late <late_lib_path> -early <early_lib_path> -help ""}
	while {[llength $args]} {
		switch -glob -- [lindex $args 0] {
		-late {
			set args [lassign $args - options(-late) ]
			puts "set_late_celllib_fpath $options(-late)"
		      }
		-early {
			set args [lassign $args - options(-early) ]
			puts "set_early_celllib_fpath $options(-early)"
		       }
		-help {
			set args [lassign $args - options(-help) ]
			puts "Usage: read_lib -late <late_lib_path> -early <early_lib_path>"
			puts "-late <provide late library path>"
			puts "-early <provide early library path>"
			puts "-help - Provides user deatails on how to use the command"
		      }	
		default break
		}
	}
}
```
4. read_verilog.proc
```
proc read_verilog {arg1} {
puts "set_verilog_fpath $arg1"
}

```
5. read_sdc.proc
```
proc read_sdc {arg1} {
set sdc_dirname [file dirname $arg1]
set sdc_filename [lindex [split [file tail $arg1] .] 0 ]
set sdc [open $arg1 r]
set tmp_file [open /tmp/1 w]
puts -nonewline $tmp_file [string map {"\[" "" "\]" " "} [read $sdc]]     
close $tmp_file
#-----------------------------------------------------------------------------#
#----------------converting create_clock constraints--------------------------#
#-----------------------------------------------------------------------------#
set tmp_file [open /tmp/1 r]
set timing_file [open /tmp/3 w]
set lines [split [read $tmp_file] "\n"]
set find_clocks [lsearch -all -inline $lines "create_clock*"]
foreach elem $find_clocks {
	set clock_port_name [lindex $elem [expr {[lsearch $elem "get_ports"]+1}]]
	set clock_period [lindex $elem [expr {[lsearch $elem "-period"]+1}]]
	set duty_cycle [expr {100 - [expr {[lindex [lindex $elem [expr {[lsearch $elem "-waveform"]+1}]] 1]*100/$clock_period}]}]
	puts $timing_file "clock $clock_port_name $clock_period $duty_cycle"
	}
close $tmp_file
#-----------------------------------------------------------------------------#
#----------------converting set_clock_latency constraints---------------------#
#-----------------------------------------------------------------------------#
set find_keyword [lsearch -all -inline $lines "set_clock_latency*"]
set tmp2_file [open /tmp/2 w]
set new_port_name ""
foreach elem $find_keyword {
        set port_name [lindex $elem [expr {[lsearch $elem "get_clocks"]+1}]]
	if {![string match $new_port_name $port_name]} {
        	set new_port_name $port_name
        	set delays_list [lsearch -all -inline $find_keyword [join [list "*" " " $port_name " " "*"] ""]]
        	set delay_value ""
        	foreach new_elem $delays_list {
        		set port_index [lsearch $new_elem "get_clocks"]
        		lappend delay_value [lindex $new_elem [expr {$port_index-1}]]
        	}
		puts -nonewline $tmp2_file "\nat $port_name $delay_value"
	}
}
close $tmp2_file
set tmp2_file [open /tmp/2 r]
puts -nonewline $timing_file [read $tmp2_file]
close $tmp2_file
#-----------------------------------------------------------------------------#
#----------------converting set_clock_transition constraints------------------#
#-----------------------------------------------------------------------------#
set find_keyword [lsearch -all -inline $lines "set_clock_transition*"]
set tmp2_file [open /tmp/2 w]
set new_port_name ""
foreach elem $find_keyword {
        set port_name [lindex $elem [expr {[lsearch $elem "get_clocks"]+1}]]
        if {![string match $new_port_name $port_name]} {
		set new_port_name $port_name
		set delays_list [lsearch -all -inline $find_keyword [join [list "*" " " $port_name " " "*"] ""]]
        	set delay_value ""
        	foreach new_elem $delays_list {
        		set port_index [lsearch $new_elem "get_clocks"]
        		lappend delay_value [lindex $new_elem [expr {$port_index-1}]]
        	}
        	puts -nonewline $tmp2_file "\nslew $port_name $delay_value"
	}
}
close $tmp2_file
set tmp2_file [open /tmp/2 r]
puts -nonewline $timing_file [read $tmp2_file]
close $tmp2_file
#-----------------------------------------------------------------------------#
#----------------converting set_input_delay constraints-----------------------#
#-----------------------------------------------------------------------------#
set find_keyword [lsearch -all -inline $lines "set_input_delay*"]
set tmp2_file [open /tmp/2 w]
set new_port_name ""
foreach elem $find_keyword {
        set port_name [lindex $elem [expr {[lsearch $elem "get_ports"]+1}]]
        if {![string match $new_port_name $port_name]} {
                set new_port_name $port_name
        	set delays_list [lsearch -all -inline $find_keyword [join [list "*" " " $port_name " " "*"] ""]]
		set delay_value ""
        	foreach new_elem $delays_list {
        		set port_index [lsearch $new_elem "get_ports"]
        		lappend delay_value [lindex $new_elem [expr {$port_index-1}]]
        	}
        	puts -nonewline $tmp2_file "\nat $port_name $delay_value"
	}
}
close $tmp2_file
set tmp2_file [open /tmp/2 r]
puts -nonewline $timing_file [read $tmp2_file]
close $tmp2_file
#-----------------------------------------------------------------------------#
#----------------converting set_input_transition constraints------------------#
#-----------------------------------------------------------------------------#
set find_keyword [lsearch -all -inline $lines "set_input_transition*"]
set tmp2_file [open /tmp/2 w]
set new_port_name ""
foreach elem $find_keyword {
        set port_name [lindex $elem [expr {[lsearch $elem "get_ports"]+1}]]
        if {![string match $new_port_name $port_name]} {
                set new_port_name $port_name
        	set delays_list [lsearch -all -inline $find_keyword [join [list "*" " " $port_name " " "*"] ""]]
        	set delay_value ""
        	foreach new_elem $delays_list {
        		set port_index [lsearch $new_elem "get_ports"]
        		lappend delay_value [lindex $new_elem [expr {$port_index-1}]]
        	}
        	puts -nonewline $tmp2_file "\nslew $port_name $delay_value"
	}
}
close $tmp2_file
set tmp2_file [open /tmp/2 r]
puts -nonewline $timing_file [read $tmp2_file]
close $tmp2_file
#-----------------------------------------------------------------------------#
#---------------converting set_output_delay constraints-----------------------#
#-----------------------------------------------------------------------------#
set find_keyword [lsearch -all -inline $lines "set_output_delay*"]
set tmp2_file [open /tmp/2 w]
set new_port_name ""
foreach elem $find_keyword {
        set port_name [lindex $elem [expr {[lsearch $elem "get_ports"]+1}]]
        if {![string match $new_port_name $port_name]} {
                set new_port_name $port_name
        	set delays_list [lsearch -all -inline $find_keyword [join [list "*" " " $port_name " " "*"] ""]]
        	set delay_value ""
        	foreach new_elem $delays_list {
        		set port_index [lsearch $new_elem "get_ports"]
        		lappend delay_value [lindex $new_elem [expr {$port_index-1}]]
        	}
        	puts -nonewline $tmp2_file "\nrat $port_name $delay_value"
	}
}

close $tmp2_file
set tmp2_file [open /tmp/2 r]
puts -nonewline $timing_file [read $tmp2_file]
close $tmp2_file
#-----------------------------------------------------------------------------#
#-------------------converting set_load constraints---------------------------#
#-----------------------------------------------------------------------------#
set find_keyword [lsearch -all -inline $lines "set_load*"]
set tmp2_file [open /tmp/2 w]
set new_port_name ""
foreach elem $find_keyword {
        set port_name [lindex $elem [expr {[lsearch $elem "get_ports"]+1}]]
        if {![string match $new_port_name $port_name]} {
                set new_port_name $port_name
        	set delays_list [lsearch -all -inline $find_keyword [join [list "*" " " $port_name " " "*" ] ""]]
        	set delay_value ""
        	foreach new_elem $delays_list {
        	set port_index [lsearch $new_elem "get_ports"]
        	lappend delay_value [lindex $new_elem [expr {$port_index-1}]]
        	}
        	puts -nonewline $timing_file "\nload $port_name $delay_value"
	}
}
close $tmp2_file
set tmp2_file [open /tmp/2 r]
puts -nonewline $timing_file  [read $tmp2_file]
close $tmp2_file
#-----------------------------------------------------------------------------#
close $timing_file
set ot_timing_file [open $sdc_dirname/$sdc_filename.timing w]
set timing_file [open /tmp/3 r]
while {[gets $timing_file line] != -1} {
        if {[regexp -all -- {\*} $line]} {
                set bussed [lindex [lindex [split $line "*"] 0] 1]
                set final_synth_netlist [open $sdc_dirname/$sdc_filename.final.synth.v r]
                while {[gets $final_synth_netlist line2] != -1 } {
                        if {[regexp -all -- $bussed $line2] && [regexp -all -- {input} $line2] && ![string match "" $line]} {
                        puts -nonewline $ot_timing_file "\n[lindex [lindex [split $line "*"] 0 ] 0 ] [lindex [lindex [split $line2 ";"] 0 ] 1 ] [lindex [split $line "*"] 1 ]"
                        } elseif {[regexp -all -- $bussed $line2] && [regexp -all -- {output} $line2] && ![string match "" $line]} {
                        puts -nonewline $ot_timing_file "\n[lindex [lindex [split $line "*"] 0 ] 0 ] [lindex [lindex [split $line2 ";"] 0 ] 1 ] [lindex [split $line "*"] 1 ]"
                        }
                }
        } else {
        puts -nonewline $ot_timing_file  "\n$line"
        }
}
close $timing_file
puts "set_timing_fpath $sdc_dirname/$sdc_filename.timing"
}

```

**Using the procs to write timing files**

Check the below code tclbox.tcl

```
############################################# Calling procs needed to generate .timing file ###################################################
# Procs are used below 
puts "\nInfo: Timing Analysis Started...."
puts "\nInfo: Initializing number of threads, libraries, sdc, verilog netlist path..."
puts " Invoking required procs"
puts "reopenStdout.proc \nset_multi_cpu_usage,proc \nread_lib.proc \nread_verilog.proc \nread_sdc.prc"
source /home/vsduser/vsdsynth/procs/reopenStdout.proc
source /home/vsduser/vsdsynth/procs/set_multi_cpu_usage.proc
source /home/vsduser/vsdsynth/procs/read_lib.proc
source /home/vsduser/vsdsynth/procs/read_verilog.proc
source /home/vsduser/vsdsynth/procs/read_sdc.proc
# Writing command required for OpenTimer tool to .conf file by closing and redirecting 'stdout' to a file
reopenStdout $Output_Directory/$Design_Name.conf
#set_multi_cpu_usage -localCpu 4
read_lib -early $Early_Library_Path
read_lib -late $Late_Library_Path
read_verilog $Output_Directory/$Design_Name.final.synth.v
read_sdc $Output_Directory/$Design_Name.sdc
# Reopening 'stdout' to bring back screen log
reopenStdout /dev/tty
# Closing .conf file opened by 'reopenStdout' proc
#close $Output_Directory/$Design_Name.conf
#puts "closed .conf and redirected to stdout"
```

**Preparation of .CONF and SPEF file for OpenTimer STA**

Check the below code for the following 

```
################################################ SPEF and CONF creation #########################################################################
set enable_prelayout_timing 1
if {$enable_prelayout_timing == 1} {
	puts "\nInfo: enable_prelayout_timing is $enable_prelayout_timing. Enabling zero-wire load parasitics"
	set spef_file [open $Output_Directory/$Design_Name.spef w]
	puts $spef_file "*SPEF \"IEEE 1481-1998\" "
	puts $spef_file "*DESIGN \"$Design_Name\" "
	puts $spef_file "*DATE \"[clock format [clock seconds] -format {%a %b %d %I:%M:%S %Y}]\" "
	puts $spef_file "*VENDOR \"TAU 2015 Contest\" "
	puts $spef_file "*PROGRAM \"Benchmark Parasitic Generator\" "
	puts $spef_file "*VERSION \"0.0\" "
	puts $spef_file "*DESIGN_FLOW \"NETLIST_TYPE_VERILOG\" "
	puts $spef_file "*DIVIDER / "
	puts $spef_file "*DELIMITER : "
	puts $spef_file "*BUS_DELIMITER \[ \] "
	puts $spef_file "*T_UNIT 1 PS "
	puts $spef_file "*C_UNIT 1 FF "
	puts $spef_file "*R_UNIT 1 KOHM "
	puts $spef_file "*L_UNIT 1 UH "
	close $spef_file
}
# Appending to .conf file
set conf_file [open $Output_Directory/$Design_Name.conf a]
puts $conf_file "set_spef_fpath $Output_Directory/$Design_Name.spef"
puts $conf_file "init_timer "
puts $conf_file "report_timer "
puts $conf_file "report_wns "
puts $conf_file "report_worst_paths -numPaths 10000 "
close $conf_file
```

</details>

<details>
<summary>Day 5 P2</summary>

**Running STA and generating the QOR**

Check the below code for the update  tclbox.tcl


```
################################# Starting Timing Analysis ##########################################################
# Running STA on OpenTimer and dumping log to .results and capturing runtime
set tcl_precision 3
set time_elapsed_in_us [time {exec /home/vsduser/OpenTimer-1.0.5/bin/OpenTimer < $Output_Directory/$Design_Name.conf >& $Output_Directory/$Design_Name.results}]
set time_elapsed_in_sec "[expr {[lindex $time_elapsed_in_us 0]/1000000}]sec"
puts "\nInfo: STA finished in $time_elapsed_in_sec seconds"
puts "\nInfo: Refer to $Output_Directory/$Design_Name.results for warnings and errors"
```

**Data extraction and QOR**

Code taken form tclbox.tcl

```
# Find worst output violation
set worst_RAT_slack "-"
set report_file [open $Output_Directory/$Design_Name.results r]
set pattern {RAT}
while { [gets $report_file line] != -1 } {
	if {[regexp $pattern $line]} {
		set worst_RAT_slack "[expr {[lindex $line 3]/1000}]ns"
		break
	} else {
		continue
	}
}
close $report_file
# Find number of output violation
set report_file [open $Output_Directory/$Design_Name.results r]
set count 0
while { [gets $report_file line] != -1 } {
	incr count [regexp -all -- $pattern $line]
}
set Number_output_violations $count
close $report_file
# Find worst setup violation
set worst_negative_setup_slack "-"
set report_file [open $Output_Directory/$Design_Name.results r]
set pattern {Setup}
while { [gets $report_file line] != -1 } {
	if {[regexp $pattern $line]} {
		set worst_negative_setup_slack "[expr {[lindex $line 3]/1000}]ns"
		break
	} else {
		continue
	}
}
close $report_file
# Find number of setup violation
set report_file [open $Output_Directory/$Design_Name.results r]
set count 0
while { [gets $report_file line] != -1 } {
	incr count [regexp -all -- $pattern $line]
}
set Number_of_setup_violations $count
close $report_file
# Find worst hold violation
set worst_negative_hold_slack "-"
set report_file [open $Output_Directory/$Design_Name.results r]
set pattern {Hold}
while { [gets $report_file line] != -1 } {
	if {[regexp $pattern $line]} {
		set worst_negative_hold_slack "[expr {[lindex $line 3]/1000}]ns"
		break
	} else {
		continue
	}
}
close $report_file
# Find number of hold violation
set report_file [open $Output_Directory/$Design_Name.results r]
set count 0
while {[gets $report_file line] != -1} {
	incr count [regexp -all -- $pattern $line]
}
set Number_of_hold_violations $count
close $report_file
# Find number of instance
set pattern {Num of gates}
set report_file [open $Output_Directory/$Design_Name.results r]
while {[gets $report_file line] != -1} {
	if {[regexp -all -- $pattern $line]} {
		set Instance_count [lindex [join $line " "] 4 ]
		break
	} else {
		continue
	}
}
close $report_file
# Capturing end time of the script
set end_time [clock clicks -microseconds]
# Setting total script runtime to 'time_elapsed_in_sec' variable
set time_elapsed_in_sec "[expr {($end_time-$start_time)/1000000}]sec"
puts "\nInfo: Design Name = $Design_Name"
puts "\nInfo: Worst RAT slack = $worst_RAT_slack"
puts "\nInfo: Number of output violations = $Number_output_violations"
puts "\nInfo: Worst negative setup slack = $worst_negative_setup_slack"
puts "\nInfo: Number of setup violation = $Number_of_setup_violations"
puts "\nInfo: Worst Negative Hold Slack = $worst_negative_hold_slack"
puts "\nInfo: Number of Hold Violations = $Number_of_hold_violations"
puts "\nInfo: Number of Instances = $Instance_count"
puts "\nInfo: Time elapsed = $time_elapsed_in_sec"

```


**Final QOR report generation**

Source code :  tclbox.tcl

```
# Quality of Results (QoR) generation
puts "\n"
puts "                                                           ****PRELAYOUT TIMING RESULTS_TCLBOX****\n"
set formatStr {%15s%14s%21s%16s%16s%15s%15s%15s%15s}
puts [format $formatStr "-----------" "-------" "--------------" "---------" "---------" "--------" "--------" "-------" "-------"]
puts [format $formatStr "Design Name" "Runtime" "Instance Count" "WNS Setup" "FEP Setup" "WNS Hold" "FEP Hold" "WNS RAT" "FEP RAT"]
puts [format $formatStr "-----------" "-------" "--------------" "---------" "---------" "--------" "--------" "-------" "-------"]
foreach design_name $Design_Name runtime $time_elapsed_in_sec instance_count $Instance_count wns_setup $worst_negative_setup_slack fep_setup $Number_of_setup_violations wns_hold $worst_negative_hold_slack fep_hold $Number_of_hold_violations wns_rat $worst_RAT_slack fep_rat $Number_output_violations {
	puts [format $formatStr $design_name $runtime $instance_count $wns_setup $fep_setup $wns_hold $fep_hold $wns_rat $fep_rat]
}
puts [format $formatStr "-----------" "-------" "--------------" "---------" "---------" "--------" "--------" "-------" "-------"]
puts "\n"
```

</details>


## Day 31: Low Power Design 

<details>
<summary>Module1</summary>

**Why Low Power Design:**

+  Differentiating  "power" and "energy" and it's impact on performance:

1. *Power*:
   - Definition: Power is the rate at which energy is transferred or converted. It is the amount of energy transferred or converted per unit of time.
   - Formula: In electrical terms, power (P) is calculated as the product of voltage (V) and current (I) : P=V×I
   - As power increase following observations can be made
        * Heat dissipation increases.
        * Cooling cost increases.
        * Frequency get limited.
        * Overall material degrades faster.

2. *Energy*:
   - Definition: Energy is the capacity to do work or produce heat. It exists in various forms such as electrical, mechanical, thermal, etc.
   - Formula: In electrical terms, energy (E) can be calculated by multiplying power (P) by time (t): E=Pxt or E=VxIxt

+ Economics of power/energy:
   - Performance.
   - Cost.
        * Packaging
        * Battery capacity
        * Shipping
   - Weight.
   - Form factor
   - Functionality.
   - Context of use.
   - Comfort/Safety.

+ Low power designs are essential for various electronic devices and systems for several reasons:

1. **Battery Life**: Many devices, especially portable ones like smartphones, wearables, and IoT devices, rely on batteries. Low power designs help extend the battery life, allowing devices to operate longer without needing a recharge.

2. **Heat Dissipation**: High power consumption generates heat, which can degrade the performance and lifespan of electronic components. Low power designs reduce heat dissipation, leading to more reliable and durable devices.

3. **Environmental Impact**: Energy efficiency is crucial for reducing the environmental impact of electronic devices. Lower power consumption means less energy usage, which translates to reduced greenhouse gas emissions.

4. **Cost Reduction**: With less power consumption, devices may use smaller batteries or require less frequent charging. This can lower production costs and increase the overall affordability of the device.

5. **Portability and Mobility**: Low power designs enable smaller form factors and lighter devices, which is crucial for portable electronics. For example, in the case of wearables or medical devices, minimizing power consumption is vital for user comfort and convenience.

6. **Reliability**: Lower power consumption often leads to improved device reliability. Components operating at lower temperatures tend to have longer lifespans and reduced failure rates.

7. **Regulatory Compliance**: Many regions have regulations and standards regarding energy consumption for electronic devices. Low power designs help manufacturers comply with these regulations.

**Portable vs Mobile vs Mobility**

+ In the realm of low-power IC design, the terms "portable," "mobile," and "mobility" are often used to describe different categories or aspects related to devices and their usage.
+ Portable:
  - Portable refer to gadgets or tools that can be easily carried or moved from one place to another.
  - Low-power IC design is crucial for portable devices as they often rely on batteries.
  - Minimizing power consumption ensures longer battery life, making these devices more practical and convenient for users. Examples include laptops, tablets, portable gaming consoles, etc.
+ Mobile:
   - Mobile devices are specifically designed for use while being in motion or while on the go. They provide communication, entertainment, or productivity capabilities and often rely on wireless connectivity.
   - Mobile devices, such as smartphones, smartwatches, and GPS units, heavily benefit from low-power IC design. These devices require energy-efficient components to support functionalities like constant connectivity, GPS tracking, sensors, and multimedia capabilities while ensuring prolonged battery life.
+ Mobility:
   - Mobility in IC design refers to the ability to create integrated circuits that cater to the dynamic and changing requirements of portable and mobile devices.
   - Design is focused on creating ICs that not only consume minimal power but also offer the necessary performance and functionality demanded by portable and mobile devices. This involves designing chips that optimize power consumption during various operating modes, including active use, standby, and sleep modes, to support the mobility and versatility required by modern devices.

+ Power Management Techniques:
   - Basic Techniques: Clock gating and Multi-Threshold.
   - Advanced Techniques: MTCMOS power gating, Power gating with state retention, DVFS and Low VDD StandBy. 

</details>

<details>
<summary>Module2</summary>

**Low Power Fundamentals:**

+ Creating low-power designs involves a combination of strategies, methodologies, and techniques across various levels of IC and system design. Here are the essential aspects and practices in low-power design:

 1. **Design Goals and Requirements Analysis**:
   - **Power Budgeting**: Determine acceptable power consumption limits for different parts of the system.
   - **Use Case Analysis**: Understand different usage scenarios to optimize power usage during various modes of operation.

 2. **Architecture-Level Strategies**:
   - **Power-Aware Architectures**: Design with power efficiency in mind, utilizing techniques like clock gating, power gating, and voltage/frequency scaling.
   - **Partitioning and Power Domains**: Divide the system into power domains, enabling selective activation/deactivation of parts to conserve power.

 3. **Circuit-Level Techniques**:
   - **Transistor Level Optimization**: Use low-leakage transistors, sub-threshold operation, and other techniques to minimize leakage currents.
   - **Clock and Data Management**: Implement clock gating, data encoding, and other logic techniques to reduce dynamic power consumption.

 4. **System-Level Strategies**:
   - **Dynamic Power Management**: Employ techniques like DVFS (Dynamic Voltage and Frequency Scaling) and AVS (Adaptive Voltage Scaling) to adjust power according to workload.
   - **Low-Power Modes**: Utilize sleep, idle, or power-down modes during periods of inactivity.

 5. **Verification and Validation**:
   - **Power-Aware Simulation and Verification**: Use specialized tools and methodologies to validate power consumption estimations and optimize power-critical paths.
   - **Hardware/Software Co-Design**: Collaborate on power optimization between hardware and software to maximize efficiency.

 6. **Technological Innovations**:
   - **Advanced Process Nodes**: Benefit from newer process technologies that inherently offer better power efficiency.
   - **Emerging Design Methodologies**: Explore novel design methodologies like approximate computing, probabilistic computing, or energy harvesting for specific applications.

 7. **Tools and Methodologies**:
   - **Power Analysis Tools**: Utilize simulation tools that provide accurate power estimates at different design stages.
   - **Power-Aware Synthesis Tools**: Employ tools that optimize circuits and architectures for reduced power consumption.

 8. **Standard Compliance and Optimization**:
   - **Compliance with Power Standards**: Ensure designs meet regulatory standards for power consumption.
   - **Trade-off Analysis**: Balance performance, area, and power to achieve optimal design results.

 9. **Documentation and Knowledge Sharing**:
   - **Document Power Considerations**: Maintain comprehensive documentation detailing power design decisions, methodologies, and trade-offs.
   - **Knowledge Sharing**: Encourage knowledge sharing among design teams to propagate best practices and lessons learned.

+ Power Consumption View of SoC


+ Balance between power management and low power design.


* *Density and Delivery*
* Density:
  + Density = power/area.
  + Heat is a primary consequence of density.
  + Power consumed is a function of current junction temperature.
* Delivery: 
  + Delivery is managing I and dI/dt.
  + Power supply must supply Imax within Vmin and Vmax.
  + Power supply must withstand Imax-Imin scenario within Vmin-Vmax.

+ *Leakage and Lifetime(Reliabilty).*
+ Leakage:
   * Leakage = tax paid by transistors when they are powered on.
   * Leakage power can be enormous.
   * Leakage increses with gate size and temperature.
   * Leakage can heat up the chip.
   * Turning off is the best way to arrest leakage power.
+ Reliabilty:
   * Current degrades material.
   * High current drawn fuses material.

**Voltage Control Techniques:**

1. **Power gating** 

+ How Power Gating Works:

1. **Isolation Transistors**:
   - Power gating involves the use of isolation transistors to disconnect the power supply from inactive blocks or sections of the chip. These transistors act as switches, completely cutting off power when the block is powered down.

2. **Control Logic**:
   - Control logic determines when to enable or disable the power gates based on activity or inactivity of the specific block. It ensures that power is gated off when the block is idle and restores power when it needs to become operational.

3. **Retention Elements**:
   - To preserve critical data or state information when the block is powered down, retention elements (such as flip-flops with isolated power supplies) are often used to maintain the data during power-off periods.

+ Benefits of Power Gating:

1. **Reduction in Leakage Power**:
   - By disconnecting power to inactive blocks, leakage current flowing through transistors in those sections is significantly reduced, minimizing static power consumption.

2. **Improved Energy Efficiency**:
   - Power gating contributes to overall energy efficiency in the system, especially in battery-powered devices, by conserving power when not actively in use.

3. **Enhanced Battery Life**:
   - Extending the battery life of portable devices by minimizing power consumption during idle or standby modes through efficient power gating techniques.

4. **Heat Reduction**:
   - Shutting off power to inactive blocks reduces heat dissipation, contributing to a cooler operating temperature for the IC.

+ Challenges and Considerations:

1. **Design Complexity**:
   - Implementing power gating adds complexity to the design, requiring additional control logic and ensuring proper synchronization to avoid issues such as glitches or improper power-up sequences.

2. **Switching Overhead**:
   - Switching power gates on and off introduces some overhead in terms of delay and power consumption associated with the control logic.

3. **Design Verification**:
   - Proper verification and testing are crucial to ensure correct functionality of power gating to prevent issues like data loss or corruption during power transitions.



*Dynamic Voltage Scaling(DVS)*

+ Dynamic Voltage Scaling , also known as Dynamic Voltage and Frequency Scaling (DVFS), is a technique used to adjust the operating voltage and frequency of a processor or system dynamically based on the workload or processing requirements. DVS is employed to reduce power consumption without sacrificing performance unnecessarily. Here's an overview of Dynamic Voltage Scaling in low-power IC design:



+ How Dynamic Voltage Scaling Works:
  1. **Voltage-Frequency Relationship**:
   - DVS leverages the relationship between voltage and frequency, where lowering the operating voltage can allow for a reduction in clock frequency, thus reducing power consumption.
  2. **Operating Point Adjustment**:
   - The system or processor continuously monitors the workload or performance requirements.
   - When the workload is low, the voltage and frequency can be scaled down to operate at a lower power mode.
   - Conversely, during periods of high demand, voltage and frequency can be increased to maintain performance.
  3. **Control Mechanisms**:
   - DVS involves control mechanisms within the processor or system, including hardware-based voltage regulators, clock controllers, and software-based algorithms.
   - These mechanisms dynamically adjust voltage and frequency based on feedback from workload sensors or software instructions.

+ Benefits of Dynamic Voltage Scaling:
  1. **Power Savings**:
   - Reducing the voltage and frequency during periods of low activity or light workload significantly lowers power consumption, conserving energy and extending battery life in mobile devices.
  2. **Thermal Management**:
   - By scaling down voltage and frequency, heat generation is reduced, contributing to better thermal management and preventing overheating in the system.
  3. **Performance-Per-Watt Improvement**:
   - DVS enables better optimization of performance-per-watt by dynamically adjusting power to match the required performance levels, enhancing efficiency.

+ Challenges and Considerations:
  1. **Trade-off between Power and Performance**:
   - There might be trade-offs between power savings and performance. Aggressive voltage scaling might impact performance, causing slowdowns during high-demand tasks.
  2. **Complexity and Overhead**:
   - Implementing DVS requires complex control mechanisms and algorithms, adding overhead in terms of design complexity and validation.
  3. **Voltage-Reliability Trade-offs**:
   - Extreme voltage scaling might compromise reliability due to potential issues such as timing violations or transient faults.

+ Variations and Advanced Techniques:
  1. **Adaptive Voltage Scaling (AVS)**:
   - AVS dynamically adjusts voltage levels at a finer granularity, targeting specific functional blocks or components within the IC.
  2. **Multi-Level Voltage Scaling**:
   - Some systems employ multiple voltage and frequency levels, allowing for more nuanced adjustments based on workload characteristics.

*Low VDD StandBy*



"Low VDD standby" refers to a low voltage state applied to the power supply (VDD) of a semiconductor device, particularly during standby or idle modes. This technique is commonly employed in low-power designs to minimize power consumption when the device is in a low-activity or idle state. By reducing the voltage supplied to the IC during standby, it helps in conserving energy and extending battery life in portable devices. 

+ Key Aspects of Low VDD Standby:

  1. **Standby Power Reduction**:
   - During periods of inactivity or low usage, the device transitions into a standby mode where non-essential components or circuits are put into a low-power state by lowering the VDD voltage.
  2. **Retention of Critical Data**:
   - To ensure that critical data or state information is not lost during the low VDD standby state, retention elements or specialized memory circuits are used to preserve the data.
  3. **Transitioning Between Active and Standby States**:
   - The device should seamlessly transition between active and standby states while ensuring that the critical functionalities are preserved and can be quickly resumed when required.
  4. **Control and Monitoring Mechanisms**:
   - Control logic within the device manages the transition to and from the low VDD standby state based on system activity, ensuring proper operation and functionality when returning to an active state.

+ Benefits and Considerations:

  1. **Power Savings**:
   - Low VDD standby significantly reduces power consumption during idle periods, extending battery life in portable devices and contributing to overall energy efficiency.
  2. **Fast Wake-up and Resume**:
   - Devices utilizing low VDD standby aim to achieve quick wake-up times to swiftly transition back to an active state, ensuring minimal disruption to user experience.
  3. **Data Integrity and Retention**:
   - It is crucial to maintain data integrity and retain critical information while the device is in the low-power standby mode to resume operations seamlessly.
  4. **Design Complexity and Verification**:
   - Implementing low VDD standby involves designing robust control logic and retention elements, increasing the complexity of the design and requiring thorough verification to ensure proper functionality.
  5. **Voltage-Reliability Trade-offs**:
   - Aggressive voltage reduction in standby states might impact reliability due to potential issues such as data corruption or timing violations.

+ State retention and UPF (Unified Power Format) are two interconnected concepts in low-power  design, particularly in the context of power management techniques used to reduce power consumption in electronic devices. 
+ State Retention:
  - State retention refers to the preservation of critical data or state information within an integrated circuit during power-down or low-power modes. When a portion of a chip enters a low-power state or is powered off, it's crucial to retain specific data in certain memory elements or registers to ensure that the device can quickly resume operation without losing essential information.
  - For example, in low-power design, certain registers might store critical system configurations, context information, or data that needs to be retained during standby or power-off modes. Retention elements or specialized memory cells are employed to preserve this critical data while consuming minimal power.
+ Unified Power Format (UPF):
  - Unified Power Format (UPF) is a standardized format or language used to specify low-power design intent and methodologies in electronic designs, especially for describing power intent in digital designs and ICs. UPF provides a standardized way to define power management techniques, power domains, power modes, and power control strategies within a design.
  - UPF facilitates the description of power intent at various levels of abstraction, allowing designers to specify power domains, isolation strategies, retention strategies, power states, power switches, and more. It enables the representation of the design's power architecture and how the different elements of the chip interact during different power modes.
+ UPF is utilized to describe and specify state retention requirements within a low-power design. Designers use UPF constructs to define the retention policies and strategies for preserving critical data during power-down or low-power modes. UPF allows the specification of retention registers, control signals, and power modes necessary to ensure that essential state information is retained while consuming minimal power.

</details>

<details>
<summary>Module3</summary>

**Deep Dive into state space**


+ *Power state space*
+ In low-power  design, the "power state space" refers to the various states or modes in which a semiconductor device can operate concerning power consumption. Managing power consumption is critical in modern electronics, especially in portable devices, IoT (Internet of Things) devices, and other battery-powered systems.
+ The power state space typically includes different operational modes or states that an IC can transition between to optimize power usage. Some common power states in low-power design include:
  1. Active Mode: This is the typical operating state where the IC is actively performing its functions, and all circuits are functioning at full capacity. It consumes the most power.
  2. Sleep or Standby Mode: In this state, parts of the IC are powered down or put into a low-power mode while retaining some functionality. It reduces power consumption compared to active mode but allows the device to quickly return to an active state.
  3. Idle Mode: Similar to sleep mode but with a slightly higher power consumption level. In this state, the IC reduces its power consumption while remaining ready to resume full operation quickly.
  4. Power-Off Mode: This is the state where the IC is completely powered down, often used when the device is turned off or in hibernation. It consumes minimal power but requires a longer time to resume normal operation.
+ Low-power IC design involves optimizing the transitions between these power states to minimize power consumption without compromising the device's functionality or responsiveness.
+ Techniques such as power gating (isolating parts of the chip when not in use), voltage scaling (adjusting voltage levels for lower power), clock gating (stopping clock signals to inactive parts), and various design methodologies help manage the power state space effectively.
+ Designers use power management units (PMUs) or power management integrated circuits (PMICs) to control and regulate the power delivery to different sections of the chip, enabling efficient utilization of power states based on the device's requirements at any given time. Balancing performance with power consumption is crucial in low-power IC design to prolong battery life, reduce heat dissipation, and enhance overall energy efficiency.

+ *Low power DUT*



+ Low power DUTs are engineered using specialized design techniques and methodologies to reduce their power consumption while maintaining essential functionalities and performance.
+ *low power VMM(Verification Methodology Manual) testbench*


+ *LP VMM*


+ A low power VMM testbench is designed specifically for verifying and validating the functionality, performance, and power management features of low-power IC designs. It encompasses several strategies and methodologies to ensure that the design functions correctly while consuming minimal power.
   1. **Power-Aware Testbench Architecture:** The testbench architecture is modified or extended to include components that model power management units (PMUs), power domains, power modes, and transitions between different power states of the DUT.
   2. **Power-Aware Stimulus Generation:** The testbench generates stimuli that cover various power modes and transitions, ensuring the DUT behaves correctly when switching between different power states. This includes test scenarios that verify functionality during power-up, power-down, sleep modes, and transitions between active and low-power states.
   3. **Checkers for Power Consumption:** Integrating checkers or monitors within the testbench to verify power consumption levels against specified criteria. These checkers analyze power-related signals and ensure the DUT operates within acceptable power limits in different operational modes.
   4. **Power-Aware Coverage Metrics:** Defining and tracking coverage metrics that specifically target power-related scenarios and transitions. This includes coverage for different power modes exercised during testing to ensure comprehensive verification.
   5. **Simulation and Emulation Environments:** Leveraging simulation and emulation environments to validate low-power features. Emulation platforms allow for more extensive testing of power states, transitions, and interactions with software running on the DUT.
   6. **Advanced Verification Techniques:** Employing advanced techniques like assertion-based verification, formal verification, and power-aware simulation to comprehensively validate low-power features.
   7. **Scenario-Based Testing:** Creating test scenarios that stress the DUT under different power conditions to ensure proper functionality and performance across various power modes.

+ Island ordering
+ The concept of "island ordering" is an integral part of the power optimization strategy in low-power design. It involves the organization and prioritization of power domains or islands within a chip to optimize power management and reduce overall power consumption.
+ In low-power design, complex chips are often divided into multiple functional blocks or domains that can be independently controlled for power management purposes. These power domains, often referred to as islands, can be powered on or off autonomously, allowing parts of the chip to operate in different power modes.
+ Island ordering specifically refers to the sequence or hierarchy in which these power domains are powered up or down during different operational phases of the chip. The goal is to manage the power-up and power-down sequences in a way that ensures correct functionality, avoids glitches or issues, and minimizes power consumption.
   1. **Dependency and Hierarchical Structure:** Determining the dependencies and relationships between different functional blocks or islands within the chip. Some blocks may need to be powered up before others to maintain proper functionality or to avoid issues such as data corruption or signal integrity problems.
   2. **Power-Up Sequence:** Establishing the order in which the power domains or islands are powered up to ensure that essential blocks required for the chip's initial operation are activated first. This helps in initializing the chip correctly without causing functional or timing issues.
   3. **Power-Down Sequence:** Defining the sequence for powering down the islands in a way that avoids potential hazards like data loss, signal glitches, or unintended interactions between different parts of the chip.
   4. **Power Management Control:** Implementing control mechanisms and protocols to manage power state transitions efficiently. This might involve using power management units (PMUs) or dedicated hardware/software mechanisms to coordinate the sequencing of power states.
   5. **Verification and Testing:** Performing rigorous verification and testing to validate the correctness of the power sequencing and to ensure that the power domains operate as intended under different scenarios and use cases.

**Basic Multivoltage terminology**

+ In most ICs, various parts of the chip require different voltage levels for their proper operation. These voltage supply lines are commonly referred to as "rails." They provide the necessary voltages to specific sections of the chip, such as the core logic, input/output (I/O) interfaces, memory blocks, or other functional units.
+ Some common types of rails in IC design include:
   - Core Voltage Rail: This supplies power to the core logic of the chip, which includes the computational units and processing elements. It is crucial for the fundamental operation of the IC.
   - I/O Voltage Rail: Provides power to the input/output interfaces of the chip, enabling communication with external devices or other integrated circuits.
   - Memory Voltage Rail: Supplies power to the memory components (e.g., SRAM, DRAM) within the chip.
   - Analog/Digital Voltage Rails: Some ICs might have separate voltage rails for analog and digital circuits to ensure proper operation and avoid interference between these components.
+ "Multi Vdd"  is a design technique used in low power design where different sections or blocks of a chip are powered by independent and separate voltage supplies. Each voltage domain, or Vdd, operates at its designated voltage level, which may differ from other parts of the chip.
+ The rationale behind employing multiple voltage domains in IC design is to optimize power consumption, improve performance, and address specific design requirements for different sections of the chip. By utilizing varying voltage levels tailored to the needs of different functional blocks, designers can achieve several advantages:
   1. **Power Efficiency:** Various sections of the chip might have different power requirements. Using multiple voltage domains allows each section to operate at its optimal voltage level, minimizing power consumption. Low-power blocks can run at lower voltages, while high-performance sections can operate at higher voltages for increased speed.
   2. **Performance Optimization:** Critical sections or high-speed interfaces within the chip can benefit from higher voltage levels, improving performance without affecting other parts of the chip that do not require such high speeds.
   3. **Noise Isolation:** Voltage domains can provide isolation from noise or interference generated by other parts of the chip. This separation helps maintain signal integrity and reduces the impact of noise on sensitive circuits.
   4. **Reduced Leakage:** Lowering the voltage in certain sections can help decrease leakage currents, especially in idle or standby modes, contributing to overall power savings.
+ Implementing multiple voltage domains requires careful design and management:
   - **Power Management Units (PMUs):** These units are responsible for regulating and managing the various voltage levels, ensuring that each section receives the appropriate voltage while coordinating power state transitions between domains.
   - **Isolation and Level Shifting:** To prevent interference between voltage domains, isolation techniques, and level shifters are often employed to enable communication between different sections operating at distinct voltage levels.
   - **Design Verification:** Rigorous verification and testing are essential to ensure proper functionality, timing, and correct interaction between the different voltage domains, as errors or issues in voltage transitions could lead to functional failures or performance degradation.

+ MTCMOS (Multi-Threshold CMOS) power gating is a power-saving technique commonly used in low power design to reduce static power consumption in modern semiconductor devices. Static power, also known as leakage power, refers to the power dissipation that occurs even when the chip is in a standby or idle state.
+ MTCMOS power gating involves selectively shutting down power to specific sections or blocks of a chip when they are not in use, thereby reducing leakage current and overall power consumption. This technique is particularly effective in scenarios where certain parts of the chip are inactive for extended periods.
+ Here's how MTCMOS power gating typically works:
   1. **Isolation of Power Domains:** The chip is divided into multiple power domains. Each domain represents a specific section or block of the chip that can be independently powered on or off.
   2. **Controlled Power Switches:** Within each power domain, there are dedicated power switches or transistors (often high threshold voltage transistors) known as power gates or isolation cells. These gates act as switches to control the flow of power to the domain.
   3. **Power State Transition:** When a specific section of the chip is not actively in use (during idle periods or when certain functionalities are not required), the associated power gate is activated to cut off power supply to that domain. This action effectively isolates the inactive section from the rest of the chip, minimizing leakage current and reducing power consumption.
   4. **Power-Up and Power-Down Sequencing:** Before activating or deactivating a power domain, careful sequencing of power-up and power-down operations is essential to prevent glitches, maintain data integrity, and ensure proper functionality when transitioning between power states.
   5. **Control and Management Logic:** A power management unit (PMU) or control logic oversees the activation and deactivation of power gates based on the chip's operational requirements. It coordinates the transitions between different power states to manage power consumption effectively.

+ Level shifting is a process used in integrated circuit (IC) design to convert signals from one voltage level to another. This technique is essential when interfacing different parts of a circuit or connecting components operating at different voltage levels, ensuring proper communication and functionality between them.
+ There are various scenarios where level shifting is required:
   1. **Between Different Voltage Domains:** In a multi-voltage domain IC, different sections of the chip might operate at distinct voltage levels. Level shifting is necessary when signals need to pass between these domains to ensure compatibility and prevent damage to components due to voltage mismatches.
   2. **Interfacing with External Components:** When an IC needs to communicate with external devices or components operating at different voltage levels (e.g., sensors, memory devices, communication interfaces), level shifting facilitates proper signal transfer.
   3. **Mixed-Signal Circuits:** In mixed-signal designs where analog and digital circuits coexist, level shifting ensures seamless communication between the analog and digital domains, preserving signal integrity.
+ Level shifting techniques can involve various methods depending on the specific requirements and constraints of the design:
   1. **Voltage-Level Translation Gates:** Dedicated circuits or voltage-level translation gates using specialized transistors or circuitry to convert signal levels between different voltage domains.
   2. **Voltage-Level Translation Buffers:** Dedicated buffer circuits designed to accept input signals at one voltage level and produce corresponding output signals at a different voltage level.
   3. **Bi-directional Level Shifters:** Circuits capable of shifting signals bidirectionally, enabling communication between voltage domains in both directions.
   4. **Diode-Clamped or Voltage-Divider Techniques:** Simple and often used for lower-speed or less critical applications, employing diodes or resistive networks to shift signal levels.
   5. **Specialized Level-Shifting ICs:** Dedicated integrated circuits specifically designed for level shifting applications, offering multiple channels and optimized performance for voltage translation.

</details>


<details>

<summary>Module4</summary>

+ ARM-based System-on-Chips (SoCs) are widely used in various devices, including smartphones, tablets, IoT devices, embedded systems, and more. Power management is crucial in these systems to optimize performance, extend battery life, and manage thermal constraints. Several common power management schemes are implemented in ARM-based SoCs:
   1. **Dynamic Voltage and Frequency Scaling (DVFS):** DVFS is a technique that adjusts the voltage and frequency of the CPU based on the workload. It dynamically scales the CPU frequency and voltage to match the processing demands. When the workload is low, the frequency and voltage are decreased to save power, while they are increased during higher workload periods to enhance performance.
   2. **CPU Power Modes (Idle States):** ARM-based SoCs often implement multiple power states for CPUs. These power modes, such as idle or sleep states (e.g., C-states in ARM's terminology), allow parts of the CPU to enter low-power states when not actively processing tasks. During idle times, certain parts of the CPU are powered down or operate at reduced frequencies to conserve power.
   3.  **Heterogeneous Multi-Processing (HMP):** HMP architectures in ARM SoCs enable the use of multiple types of CPU cores with varying performance and power characteristics. This scheme dynamically assigns tasks to different cores based on their power-performance trade-offs. Lower-power cores handle less demanding tasks while higher-performance cores manage more intensive tasks, optimizing power usage.
   4.  **Peripheral Power Management:** ARM-based SoCs include various peripherals, such as GPUs, DSPs, and I/O controllers. Power management techniques like clock gating, power gating, and dynamic power scaling are applied to these peripherals. By selectively enabling or disabling peripherals and adjusting their operating voltages or frequencies, power consumption can be reduced.
   5.  **Adaptive Voltage Scaling (AVS):** AVS adjusts the voltage levels supplied to different components based on the required performance. It dynamically scales the voltage to the lowest level that still maintains stable operation, reducing power consumption without sacrificing performance.
   6.  **Temperature and Thermal Management:** ARM SoCs often incorporate thermal management schemes to monitor and regulate temperature. Dynamic thermal management techniques, such as throttling or reducing processor speed in response to elevated temperatures, help prevent overheating while maintaining operational stability.
   7.  **Software-Based Power Governors:** Power governors within the operating system or firmware of ARM-based devices manage and control power states, determining when to transition between different power modes based on system demands and user settings.
+ These power management schemes, often implemented in combination, aim to balance performance and power consumption in ARM-based SoCs, providing efficient and responsive devices while optimizing energy usage and extending battery life.

+ *Power Management Brings New Bug Types!*
   -  Isolation/Level Shifting Bugs
   -  Control Sequencing bugs
   -  Retention scheme/control errors
   - Retention selection errors
   - Electrical Problems like memory corruption
   - Power Sequencing/Voltage Scheduling errors
   - Hardware-Software deadlock
   - Power Gating collapse/dysfunction
   - Power On Reset/bring up problems
   - Thermal runaway/ Overheating

+ *Conflicting Events:*
+ Conflicting events in low-power design power management refer to scenarios or situations where different power-saving mechanisms or requirements clash, causing conflicts or challenges in managing power efficiently. These conflicts can arise due to conflicting requirements between various power-saving techniques or constraints within the design.
   1. **Voltage-Frequency Conflicts:** While dynamic voltage and frequency scaling (DVFS) aim to reduce power consumption by lowering voltage and frequency during low activity, high-performance demands may conflict with this strategy. For instance, an application requiring high performance might clash with the goal of reducing voltage and frequency to save power, creating a conflict between performance and power efficiency.
   2. **Clock Gating vs. Timing Requirements:** Clock gating is a technique used to stop clock signals to inactive blocks to save power. However, certain blocks may have strict timing requirements or dependencies that conflict with the idea of gating their clocks. Balancing power savings with meeting timing requirements can create conflicts.
   3. **Power Gating and Wake-up Time:** Power gating involves shutting down power to inactive blocks. However, the time it takes to power up these blocks and return to full operation (wake-up time) might conflict with the need for instant responsiveness in some applications. This conflict arises between power savings and responsiveness.
   4. **Multiple Power Domains Coordination:** Managing multiple voltage or power domains within a chip can create conflicts when transitioning between power states. Timing and sequencing requirements for turning on or off different domains may conflict with overall system operation, leading to potential glitches or errors during transitions.
   5. **Trade-offs between Power and Functionality:** Certain power-saving techniques might compromise the functionality or performance of the system. For instance, overly aggressive power-saving methods might result in degraded system performance, creating a conflict between power efficiency and functional requirements.
+ Resolving conflicting events in low-power design power management involves careful trade-offs, optimizations, and compromises. Designers need to consider the specific requirements of the system, application use cases, and the balance between power-saving strategies and the overall functionality or performance goals to mitigate these conflicts and achieve an optimal balance between power efficiency and system operation.
+ Combining Multiple CPUs
  - Must take a hierarchical sub-system view
  - Must be conscious of s/w threads and h/w events in each sub-system
  - An "FSM" view of power states must be commonly known across the sub-system boundary
  - E.g ACPI
+ Best to enforce a consistent protocol across all sub-systems
+ Industry standards are only emerging here
+ Homogeneous subsystems make code-reuse possible
+ A draw back of heterogeneous subsystems, but this is common

+ *Power Management verification:*
+ Power management verification in a design involves validating and ensuring that the implemented power-saving features and techniques operate correctly and efficiently within an IC. This verification process ensures that the power management strategies effectively reduce power consumption without compromising the functionality, performance, or reliability of the IC.
   1. **Functional Verification:** This involves verifying that the power management features function as intended. It includes validating power state transitions, power-on and power-off sequences, handling of different power modes, and ensuring proper functioning of power control logic.
   2. **Simulation and Emulation:** Employing simulation tools and emulation platforms to model and simulate power states and transitions within the IC. Simulations validate the behavior of the power management circuits under various conditions and use cases.
   3. **Coverage Analysis:** Defining and measuring coverage metrics specific to power management scenarios. Coverage analysis ensures that the verification tests cover a comprehensive range of power states, transitions, and functional behaviors related to power management.
   4. **Assertion-Based Verification:** Writing assertions to check and validate specific power-related conditions or behaviors within the design. Assertions act as checks to ensure that power management protocols are followed correctly during simulation or emulation.
   5. **Formal Verification:** Using formal methods to mathematically verify the correctness of power management implementations against specified requirements or properties. Formal verification can help detect potential power-related issues or design flaws.
   6. **Low-Power Design Verification Tools:** Leveraging specialized verification tools tailored for low-power designs. These tools assist in verifying complex power management architectures, identifying power-related issues, and optimizing power-saving strategies.
   7. **Hardware Emulation and Prototyping:** Building hardware prototypes or using emulation platforms to validate power management strategies in real-world scenarios. Hardware emulation provides a more accurate representation of power behavior and allows for extensive testing of power-related features.
   8. **Dynamic Power Analysis:** Performing power analysis during simulation or post-silicon testing to measure actual power consumption and validate against expected power budgets or specifications.
+ Power management verification is crucial in ensuring the reliability, efficiency, and correctness of power-saving features within IC designs. It helps prevent issues related to power states, transitions, and overall power control, ensuring that the IC operates optimally in terms of power consumption while meeting functional and performance requirements.


</details>

<details>
<summary>Module5</summary>

**Island Ordering:**

+ Island ordering is a technique used in low-power) design to reduce power consumption by optimizing the placement of different voltage islands on the chip.
+  Voltage islands are groups of logic blocks that operate at different supply voltages. By placing voltage islands with similar voltage levels closer together, the designer can minimize the length of the power supply wires, which reduces the voltage drop across the wires and thus the power consumption.
+ There are two main approaches to island ordering:
    1) Top-down 
    2) Bottom-up.
+ Top-down island ordering involves partitioning the chip into voltage islands based on a high-level power analysis. This approach is relatively simple to implement, but it may not be as effective as bottom-up island ordering, which involves using a more detailed power analysis to optimize the placement of individual logic blocks.
+ Bottom-up island ordering is a more complex approach, but it can potentially achieve greater power savings. This approach involves using a power grid analysis tool to identify the optimal placement of individual logic blocks. The power grid analysis tool takes into account the power consumption of each logic block, as well as the resistance and capacitance of the power supply wires.
+ Once the power grid analysis tool has identified the optimal placement of the logic blocks, the designer can then place the voltage islands on the chip. The designer should try to place voltage islands with similar voltage levels closer together, and should also try to minimize the length of the power supply wires.
+ Island ordering is a valuable technique for reducing power consumption in low-power  design. By optimizing the placement of different voltage islands on the chip, the designer can minimize the length of the power supply wires and thus the power consumption.

**Power Formats**

+ In low-power design, power formats play a crucial role in effectively managing and optimizing power consumption. These formats provide a standardized way to represent and analyze power intent, enabling designers to make informed decisions about power gating, voltage scaling, and other power-saving techniques.
+ Common Power Formats: There are several widely used power formats in low-power IC design, each with its own strengths and limitations:
   - Unified Power Format (UPF): UPF is the industry-standard power format, defined by the IEEE 1801 standard. It provides a comprehensive framework for describing power intent, including supply voltages, power domains, leakage models, and power-aware design constraints. UPF supports hierarchical power management, enabling designers to specify power intent at different levels of abstraction.
   - Power Analysis Markup Language (PAM-XML): PAM-XML is an XML-based power format developed by Mentor Graphics. It is a lightweight and easy-to-use format, specifically designed for early-stage power analysis and estimation. PAM-XML is less comprehensive than UPF but offers a simpler and more intuitive syntax.
   - Common Power Format (CPF): CPF is a power format developed by Synopsys. It is similar to UPF but provides additional features for power-aware synthesis and optimization. CPF supports power-aware clock tree synthesis, power-aware scan insertion, and other power-saving techniques.
   - PowerIntent (PI): PowerIntent is a power format developed by Cadence Design Systems. It is a newer format, gaining popularity due to its support for power-aware design in advanced process technologies. PowerIntent provides enhanced features for power modeling of leakage currents, crosstalk effects, and power grid analysis.
+ Choosing the Right Power Format: The choice of power format depends on several factors, including the design methodology, tools used, and level of power analysis detail required. UPF is the industry standard and is widely supported by EDA tools. It is well-suited for complex power management scenarios and detailed power analysis. PAM-XML is a good choice for early-stage power estimation and for designers who prefer a simpler format. CPF is specifically designed for power-aware synthesis and optimization. PowerIntent is a newer format gaining traction due to its advanced features for power modeling in advanced process technologies.
+ Benefits of Using Power Formats:
   - Standardized Representation: Power formats provide a standardized way to represent power intent, enabling designers to communicate power requirements clearly and consistently across different design teams and EDA tools.
   - Early Power Analysis: Power formats support early-stage power analysis, allowing designers to identify and address power issues early in the design cycle.
   - Power-Aware Optimization: Power formats enable power-aware optimization techniques, such as power gating, voltage scaling, and clock gating.
   - Design for Manufacturability (DFM): Power formats can be used to perform power grid analysis and ensure that the design meets power delivery requirements.
   - Design Reuse: Power formats facilitate the reuse of power intent across different designs, reducing design time and improving consistency.

**UPF**

+ Unified Power Format (UPF) is an industry-standard specification for describing power intent in integrated circuits (ICs). It provides a structured and consistent way to convey power management information, enabling designers to effectively manage and optimize power consumption in low-power designs. UPF plays a crucial role in various phases of the design process, from early-stage power estimation to physical implementation and signoff.
+ UPF offers a comprehensive set of features for describing power intent in ICs:
   - Supply Voltages: UPF defines supply voltages for different power domains, enabling efficient power distribution and minimizing power drops.
   - Power Domains: UPF allows designers to group logic blocks into power domains, enabling granular control over power management.
   - Leakage Models: UPF supports leakage models for different types of transistors, enabling accurate power estimation and optimization.
   - Power-Aware Constraints: UPF provides a mechanism to specify power-aware constraints, such as power gating and voltage scaling thresholds.
   - Hierarchical Power Management: UPF supports hierarchical power management, enabling designers to specify power intent at different levels of abstraction.
   - Tool Control Language (TCL): UPF utilizes TCL for scripting and automation, facilitating integration with various EDA tools.
+ Applications of UPF
   - Power Estimation: UPF information is used for early-stage power estimation, enabling designers to identify and address power issues early on.
   - Power Optimization: UPF guides the application of power-saving techniques, such as power gating, voltage scaling, and clock gating.
   - Physical Design: UPF is used to generate power delivery networks and ensure that the design meets power delivery requirements.
   - Design Verification: UPF-based simulations are used to verify the functionality and power behavior of the design under various operating conditions.
   - Signoff: UPF information is incorporated into signoff tools to ensure that the design meets power requirements and complies with manufacturing specifications.

+ Impact of UPF on Low-power Design
   - Promoting Standardization: UPF has established a standardized way to represent power intent, enabling seamless communication across design teams and EDA tools.
   - Enhancing Power Efficiency: UPF-based power optimization techniques have led to significant reductions in power consumption, enabling smaller and more energy-efficient designs.
   - Streamlining Design Flow: UPF has streamlined the design flow by automating power management tasks and integrating power information throughout the design process.
   - Improving Design Productivity: UPF has improved design productivity by reducing the time and effort required to manage and optimize power consumption.
 
</details>
