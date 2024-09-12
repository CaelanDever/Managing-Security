# Managing-Security

# Task: Implementing SELinux Policies for Enhanced Security

# SELinux on CentOS 8

# 1. Enable SELinux Enforcement Mode

Checking SELinux Status:

sestatus

This command checks the current status of SELinux. You’ll see whether SELinux is enabled or disabled, and what mode it’s operating in (enforcing, permissive, or disabled).

Edit SELinux Configuration File:

sudo nano /etc/selinux/config

This command opens the SELinux configuration file in the nano text editor. SELinux mode can be set to enforcing, permissive, or disabled:

enforcing - SELinux policy is enforced.
permissive - SELinux policy is not enforced, but violations are logged.
disabled - SELinux is completely turned off.

Ensure the line SELINUX=enforcing is set to enforce the SELinux policy. Save and exit the editor.

Apply Changes: If you changed the configuration to enforcing, you need to 
apply it:

sudo setenforce 1

The setenforce command changes the SELinux mode at runtime. 1 sets it to 
enforcing mode. This change is temporary and will revert after a reboot unless updated in the configuration file.

# 2. Configure SELinux Policies for /var/www/html

Install Required Tools:

sudo dnf install policycoreutils-python-utils

This command installs tools for managing SELinux policies, including semanage, which helps in managing SELinux policy components.

Check Current Context of /var/www/html:

ls -Z /var/www/html

The -Z option shows SELinux security contexts associated with the files and directories. This helps you verify the current SELinux context for 

/var/www/html.

Adjust Context (if necessary):

sudo semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"

sudo restorecon -Rv /var/www/html

semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?": Adds a 
new file context rule. httpd_sys_content_t is the context for files served by HTTPD.

restorecon -Rv /var/www/html: Applies the new context rules recursively to the directory. The -R option means recursive, and -v is verbose, showing detailed output.

# 3. Define and Create the Custom SELinux Policy Module

Install Policy Development Tools:

sudo dnf install selinux-policy-devel

Installs development tools needed to create and manage SELinux policies, including the checkmodule and semodule_package commands.

Create Policy Module:

Create a File named web_policy.te:

sudo nano web_policy.te

Opens a new text file where you define your custom policy. Add the following content to allow HTTPD access:

policy_module(web_policy, 1.0)

#Allow HTTPD to read and write within /var/www/html
allow httpd_t var_t:file { read write };

policy_module(web_policy, 1.0): Declares a new SELinux policy module named web_policy with version 1.0.

allow httpd_t var_t:file { read write };: Grants httpd_t (the type for HTTPD processes) read and write permissions on files labeled with var_t (the type for files in /var/www/html).

Compile and Install the Module:

checkmodule -M -m -o web_policy.mod web_policy.te

semodule_package -o web_policy.pp -m web_policy.mod

sudo semodule -i web_policy.pp

checkmodule -M -m -o web_policy.mod web_policy.te: Compiles the policy 
source file (web_policy.te) into a module file (web_policy.mod).

semodule_package -o web_policy.pp -m web_policy.mod: Packages the module into a policy package file (web_policy.pp).

sudo semodule -i web_policy.pp: Installs the policy package into SELinux.

# 4. Apply and Verify the SELinux Policy

Verify the Policy Module is Loaded:

semodule -l | grep web_policy

Lists all installed SELinux modules and filters the list to show if web_policy is among them.

Test HTTPD Access:

Create a test file in /var/www/html/testfile:

echo "Test" | sudo tee /var/www/html/testfile

Uses tee to create and write "Test" to testfile within /var/www/html.

Access the file via HTTPD:

curl http://localhost/testfile

Verifies that the file is accessible via HTTPD, indicating the policy allows read access.

Check SELinux Logs for Denials:

sudo ausearch -m avc -ts recent

Searches the audit logs for recent SELinux denials (avc stands for Access Vector Cache, which logs access denials).

# AppArmor on Ubuntu

# 1. Enable AppArmor

Check AppArmor Status:

sudo apparmor_status

Shows the status of AppArmor, including which profiles are loaded and their status (enforced, complain, or disabled).

Ensure AppArmor is Enabled: If AppArmor isn’t running:

sudo systemctl start apparmor
sudo systemctl enable apparmor

start initiates the AppArmor service.

enable ensures AppArmor starts at boot.

# 2. Configure AppArmor Profiles for /var/www/html

Install AppArmor Utilities:

sudo apt-get install apparmor-utils

Installs tools like aa-status, aa-enforce, and aa-complain to manage and enforce AppArmor profiles.

Create and Edit a New Profile:

Create a New AppArmor Profile for HTTPD:

sudo nano /etc/apparmor.d/usr.sbin.apache2

Edit the profile to include rules for /var/www/html:

/var/www/html/** rw,

This rule allows read and write access to all files and directories under /var/www/html for the Apache process.

Reload the AppArmor Profile:

sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.apache2
Reloads the modified profile into the AppArmor framework to apply changes.

# 3. Define and Apply the Custom AppArmor Policy

Create a Custom AppArmor Profile:

Create Profile File:

sudo nano /etc/apparmor.d/custom_web_profile

Add:

#include <tunables/global>

/var/www/html/ r,
/var/www/html/** rw,
#include <tunables/global>: Includes global tunable variables.
/var/www/html/ r,: Grants read access to the /var/www/html directory.
/var/www/html/** rw,: Grants read and write access to all files and subdirectories under /var/www/html.
Load and Enforce the Profile:

sudo apparmor_parser -r /etc/apparmor.d/custom_web_profile
Applies the new custom profile.

# 4. Test and Verify

Check the AppArmor Status of the Profile:

sudo apparmor_status
Verifies that the custom profile is loaded and enforced.

Verify HTTPD Access:

Create a test file and access it via HTTPD:

echo "Test" | sudo tee /var/www/html/testfile
curl http://localhost/testfile
Check AppArmor Logs for Denials:


sudo dmesg | grep apparmor
Checks kernel messages for any denials related to AppArmor.

# Conclusion

This detailed guide should help you understand each step involved in configuring SELinux and AppArmor for enhanced security on CentOS 8 and Ubuntu, respectively.
