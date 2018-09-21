# linux_patch_notes
Notes on how to submit patches to the linux kernel

Some notes on the process of submitting a linux patch as of August 1st, 2015
- I tried to add this to the kernel newbies wiki at the time but could not get anyone to give me access


=== Overview ===

1) download the source code
2) make a change
3) make a patch file
4) test the patch file
5) email that patch file to the appropriate maintainer.


=== Notes ===

Install Ubuntu, install git.
Set up send-email.

Use git to clone the correct repository.
- There are many linux repositories floating around the net.
- You need the one that is published by whoever maintains the code you want to modify.
- For this example, we'll use GregKH's usb repository.

Use checkpatch.pl to find a style problem.
- ./scripts/checkpatch.pl -f drivers/usb/host/ehci-hcd.c

Fix a style problem.
- vim drivers/usb/host/ehci-hcd.c

Create a new branch.
- git checkout -b myfix

Commit your changes.
- Create a good commit message. Read the log to see how people do it. You must sign off on the commit.
- If your patch is non-trivial, then you should use Coccinelle to generate it.
-- Coccinelle takes a semantic patch and generates the code change.
-- A semantic patch is like a specialized regular expression for c code.

Create a patch.
- git format-patch usb/usb-next..myfix

Check the patch.
- ./scripts/checkpatch.pl 0001-fix_style_problem.patch

Email the patch to yourself.
- git send-email --to me@me.com 0001-fix_style_problem.patch

Download the patch.

Apply the downloaded patch to the correct branch.
- git am ~/Downloads/0001-fix_style_problem.patch

Build the kernel.
- The hardest part of building the kernel is configuring it.
- So cheat and just copy the configuration you are currently using.
-- cp /boot/config-3.1 .
- use the following command to update the local name to something like "-myfix"
-- make nconfig
- make -j8
- sudo make modules_install
- sudo make install

Reboot, and pick your modified kernel from the boot menu

Test that everything is working.

Figure out who to send the patch to.
- ./scripts/get_maintainer.pl 0001-fix_style_issue.patch

Send an email to those people.
- git send-email --to maintainersemail@email.com --cc everyone@else.com --cc mailinglists@email.com 0001-fix_style_issue.patch

Wait.
- You will usually get a response in a week or two.

Fix any problems
- people will tell you if you messed up. Fix the problems and submit a new patch (append v2 to the name)

Repeat

Eventually the maintainer will accept your patch.

Celebrate.

Submit another patch.


=== How to find problems to fix ===

There are a large number of checkpatch.pl warnings to fix.
- There are also a large number of checkpatch errors.
- Another way to find problems is sparse, a static code analyzer.
-- Maybe smatch, too. And Coccinelle

find drivers/usb/core | xargs -P 8 -n 1 ./scripts/checkpatch.pl -f 2>/dev/null

make SUBDIRS=drivers/usb/ C=2 CF="-Wsparse-all"

make coccicheck MODE=report SUBDIRS=drivers/usb/ 2>cocci_err.txt 1>cocci_out.txt

UNTESTED:
make CHECK="~/path/to/smatch/smatch -p=kernel" C=1 bzImage modules | tee warns.txt
