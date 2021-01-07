# How to Run a Debug Pod
Blog Post for starting point: https://www.openshift.com/blog/rhel-coreos-compliance-scanning-in-openshift-4

It's useful to use TMUX with a split screen in order to run the `oc cp` command.

## Create a project to make it easier to find your pod when you want to copy the report back
`oc new-project node-scan`

## Specify 
`oc debug node/master01.ocp4.disconnect.blue --to-namespace=node-scan --image=ubi7`

To use the host binaries (typically not what you want)
`chroot /host`

## Check to see if you are scanning CoreOS
`cat /host/etc/system-release`

## Install the repos for OpenSCAP (ubi7 will not have default repos for these)
`curl -L http://copr.fedoraproject.org/coprs/openscapmaint/openscap-latest/repo/epel-7/openscapmaint-openscap-ltest-epel-7.repo -o  /etc/yum.repos.d/openscapmaint-openscap-latest-epel-7.repo`

## Update the new repo
`yum update -y`

## Install OpenSCAP
`yum install -y openscap openscap-utils scap-security-guide --skip-broken`

## Run a scan with the OpenSCAP in chroot mode (for RHEL8, this is from the docs)
`oscap-chroot /host/ xccdf eval \
--profile xccdf_org.ssgproject.content_profile_stig \
--fetch-remote-resources \
--report report.html \
/usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml`

## Run a scan against RHCOS
`oscap-chroot /host/ xccdf eval \
--profile xccdf_org.ssgproject.content_profile_moderate \
--fetch-remote-resources \
--report report.html \
/usr/share/xml/scap/ssg/content/ssg-rhcos4-ds.xml`

## Find the debug pod if you didn't point it to a namespace
`oc get pods -A | grep debug`

## Make your life a little easier in the below command
`oc project <project found above`

## Pull the report back from a seperate console
`oc cp $(oc get pods -n node-scan --no-headers | awk '{print $1}'):report.html report.html`

## Open the report
`xdg-open report.html`
