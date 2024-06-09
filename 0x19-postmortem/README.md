# Postmortem

## Incident Overview

Upon the release of ALX's System Engineering & DevOps project 0x19 at approximately 06:00 West African Time (WAT) in Nigeria, an isolated Ubuntu 14.04 container running an Apache web server experienced an outage. GET requests to the server resulted in 500 Internal Server Errors instead of serving the expected HTML file for a simple Holberton WordPress site.

## Debugging Process

Bug debugger Hcient Miracle (Hcient... you know, sounds almost like an ancient miracle, which is pretty fitting for a programmer's lucky fix!) encountered the issue at around 19:20 PST upon being instructed to address it.

1. **Checked Running Processes:**
   - Executed `ps aux` and confirmed that two Apache processes - one running as root and the other as www-data - were active.

2. **Verified Apache Configuration:**
   - Inspected the `sites-available` directory within `/etc/apache2/` and confirmed the web server was serving content from `/var/www/html/`.

3. **Strace on Apache Processes:**
   - Ran `strace` on the PID of the root Apache process and initiated a `curl` request to the server. The output from `strace` did not provide useful information.
   - Repeated the `strace` procedure on the PID of the www-data process. This time, the `strace` output revealed an `-1 ENOENT (No such file or directory)` error when attempting to access `/var/www/html/wp-includes/class-wp-locale.phpp`.

4. **Identified the Typo:**
   - Used Vim pattern matching to search through the files in `/var/www/html/` and identified the erroneous `.phpp` file extension in `wp-settings.php` (Line 137: `require_once( ABSPATH . WPINC . '/class-wp-locale.php' );`).

5. **Corrected the Typo:**
   - Removed the trailing 'p' from the line, correcting the file extension to `.php`.

6. **Tested the Fix:**
   - Executed another `curl` request to the server, which returned a 200 status code, confirming the issue was resolved.

7. **Automated the Fix:**
   - Created a Puppet manifest to automate the correction of this specific typo in the future.

## Summation

The root cause of the outage was a simple typo in the WordPress `wp-settings.php` file, which referred to a non-existent `class-wp-locale.phpp` file. The correction involved changing the file extension from `.phpp` to `.php`.

## Prevention

To prevent similar outages in the future:

1. **Thorough Testing:**
   - Test the application thoroughly before deployment. This error would have been detected and fixed earlier if the application had been tested properly.

2. **Status Monitoring:**
   - Implement an uptime-monitoring service such as UptimeRobot to provide immediate alerts in case of website outages.

In response to this incident, a Puppet manifest (`0-strace_is_your_friend.pp`) was created to automate the correction of any such typographical errors in the future. However, as diligent programmers, we aim to avoid such errors entirely! 😉

---

**Author: Hycient Miracle**

