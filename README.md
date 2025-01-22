sudo oscap xccdf eval \
    --profile stig \
    --results rhel8-results.xml \
    --report rhel8-report.html \
    /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml
	
sudo oscap xccdf eval \
    --profile nist-800-53 \
    --results nist-results.xml \
    --report nist-report.html \
    /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml

sudo oscap xccdf eval \
    --profile pci-dss \
    --results pci-dss-results.xml \
    --report pci-dss-report.html \
    /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml

sudo oscap xccdf eval \
    --profile standard \
    --results standard-results.xml \
    --report standard-report.html \
    /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml

	
