# ansible
ansible learning
var2.lon21
var2.bru1
var1.gla1
var1.lon1
var2.lon1
var3.lon1
var4.lon1
var1.lon1
var2.lon1
var1.nhm1
var1.edh1
var1.rtm1
var2.dub6
var3.lon1
var4.lon1
var3.lon21
var2.par1
var1.lys1
var1.gen2
var1.mad4
var2.mad4
var1.par1
var1.lil1
var2.ant1
var1.par1
var2.par1
var1.bcl2
var2.gen2
var2.prg1
var1.frf1
var2.frf1
var1.lux5
var2.dus1
var2.ber1
var2.sof3
var2.bch1
var2.zur3
var1.frf1
var2.frf1
var1.ams1
var2.ams1
var2.rtm1
var2.lil1
var1.bru1
var1.ant1
var2.lux5
var1.ams1
var2.ams1
var1.bdp1
var2.bdp1
var1.bch1
var1.waw2
var2.waw2
var1.sof3
var4.lon21
var1.bhe3
var2.bhe3
var1.lon21
var3.man2
var4.man2
var1.lds4
var1.mad4
var2.mad4
var1.mrs2
var2.mrs2
var3.mun1
var4.mun1
var2.vie1
var1.osl2
var2.osl2
var1.stk2
var2.stk2
var2.bdp1
var1.vie1
var1.prg1
var1.bdp1
var2.lds4
var1.bhe3
var2.bhe3
var1.dub6
var3.man2
var4.man2
var2.nhm1
var2.gla1
var2.edh1
var2.bcl2
var1.mrs2
var2.mrs2
var2.lys1
var3.mun1
var4.mun1
var1.osl2
var2.osl2
var1.hel1
var2.hel1
var1.stk2
var2.stk2
var2.waw2
var1.ber1
var1.ham1
var2.ham1
var1.dus1
var1.waw2
var1.ham1
var2.ham1



import csv
import subprocess
import re
import os

# File paths
device_list_file = "C:\\path\\to\\device_list.csv"  # Replace with your actual CSV file path containing devices
output_file_path = "C:\\path\\to\\routing_instances.csv"  # Replace with your desired output file path

# Initialize the output CSV file
with open(output_file_path, mode="w", newline="") as csv_file:
    fieldnames = ["Device", "Routing Instance", "Service type", "Interface", "Main Interface", "Sub interface", "Connection type"]
    writer = csv.DictWriter(csv_file, fieldnames=fieldnames)
    writer.writeheader()

# Read device list from CSV
with open(device_list_file, mode="r") as file:
    reader = csv.reader(file)
    devices = [row[0] for row in reader]

# Function to run commands on a device and capture the output
def run_command(device, command):
    result = subprocess.run(["roci", device, command], capture_output=True, text=True, shell=True)
    return result.stdout

# Process each device
for device in devices:
    print(f"Processing device: {device}")
    
    # Run the required commands on the device
    config_output = run_command(device, "show configuration routing-instances | display set | match 'interface|instance-type' | no-more")
    interface_output = run_command(device, "show interfaces descriptions | no-more")

    # Initialize variables for parsing
    instance_data = []
    
    # Parse routing instances and interfaces
    for line in config_output.strip().splitlines():
        match_instance = re.match(r"set routing-instances (\S+) instance-type (\S+)", line)
        match_interface = re.match(r"set routing-instances (\S+) interface (\S+)", line)
        
        if match_instance:
            instance_name, service_type = match_instance.groups()
        elif match_interface:
            instance_name_from_interface, interface = match_interface.groups()
            if instance_name == instance_name_from_interface:
                instance_data.append((instance_name, service_type, interface))

    # Parse interface descriptions
    interface_descriptions = {}
    for line in interface_output.strip().splitlines():
        if line.startswith("Interface"):
            continue
        parts = line.split(maxsplit=3)
        if len(parts) > 3:
            interface_name = parts[0]
            description = parts[3]
            interface_descriptions[interface_name] = description

    # Append the data to the output CSV file
    with open(output_file_path, mode="a", newline="") as csv_file:
        writer = csv.DictWriter(csv_file, fieldnames=fieldnames)

        for instance_name, service_type, interface in instance_data:
            main_interface, sub_interface = (interface.split(".", 1) + [""])[:2]
            description = interface_descriptions.get(main_interface, "")
            connection_type = description if description else ""

            writer.writerow({
                "Device": device,
                "Routing Instance": instance_name,
                "Service type": service_type,
                "Interface": interface,
                "Main Interface": main_interface,
                "Sub interface": sub_interface,
                "Connection type": connection_type,
            })

print(f"CSV file '{output_file_path}' updated successfully with data from all devices.")

