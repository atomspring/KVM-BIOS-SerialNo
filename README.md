# KVM-BIOS-SerialNo
How to provide a BIOS Serial to KVM Guests on Debian-esque systems

-= This is based on the awesome work by some guy named "war" over at https://www.undrground.org/node/79 =-

So, you've got a VM guest system with software which refuses to run because the BIOS doesn't have a serial number? (For instance, AutoCAD 2014 on a Windows 7 guest crashing due to LMU.exe and junk). This guide focuses on Ubuntu and Apparmor, for selinux take a look at war's original write-up sourced above.

Step 1: Determine if a lack of BIOS serial number is the problem. On a Windows guest VM, open cmd and run ```wmic bios get serialnumber```. On a Linux guest, run ```dmidecode -s system-serial-number``` in terminal.

If you get nothing back on the prompt, hey presto! This guide will help you. Otherwise, dang. Can't help you.

Step 2: Become root on a terminal of the host system. NOTE: I use SPICE as my KVM emulator for performance reasons, normally most people use VNC, so simply replace kvm-spice with just kvm if you're an average KVM sysadmin.

Anyway, ```nano /usr/bin/kvm-spice-serial``` and stick this in it:
```
#!/usr/bin/perl
my $qemukvm = "/usr/bin/kvm-spice";
my $args;
my $serial;
foreach (@ARGV) {
  $args .= " $_";
  if ($_ =~  m/[\w]{8}(-[\w]{4}){3}-[\w]{12}/) {
    my @tmp = split(/-/, $_);
    $serial = $tmp[0];
  }
}
if ($serial) {
  exec ($qemukvm . $args . " -smbios type=1,serial=" . $serial);
} else {
  exec ($qemukvm . $args);
}
```
Step 3: Save the file, ```chmod +x /usr/bin/kvm-spice-serial```

Step 4: Shutdown your VM, ```virsh list --all``` and then ```virsh edit <case sensitive VM name>```

Step 5: Look for a line beginning with <emulator>, replace the specified executable with /usr/bin/kvm-spice-serial. Exit, make sure virsh doesn't complain about anything.

Step 6: Update AppArmor: ```echo '/usr/bin/kvm-spice-serial rmix,' >> /etc/apparmor.d/abstractions/libvirt-qemu && service apparmor reload```

Step 7: Start the VM, and check for BIOS serial.
