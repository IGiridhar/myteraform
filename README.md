RHEL Monthly Patching
1. Prerequisite Infrastructure Setup
Mount a NFS drive as specified below on all hosts. 
•	Capacity: 500GB of NFS storage per environment.
•	Mount Point: Standardized as /repo/rhel on every server.
•	Permissions:
o	On bastion host: Mount as rw (Read-Write) to perform syncs and extractions.
o	On all DB hosts: Mount as ro (Read-Only) to prevent accidental tampering.
1.2 Detailed Directory Structure Line Diagram
The following structure must be maintained on the NFS mount to ensure that dnf can resolve the metadata for both core OS packages and modular application streams.
 
________________________________________
 
2. Phase 1: Monthly Repository Sync (QA)
Performed once a month on the QA Build Server.
1.	Prepare Variables:
•	export REL_VER="8.10" # Switch to 8.7 as needed
•	export SNAP_DATE=$(date +%Y-%m-%d)
•	export BASE_DIR="/repo/rhel/$REL_VER/snapshots/$SNAP_DATE"
•	sudo mkdir -p $BASE_DIR
2.	Sync Packages from Red Hat CDN:
•	sudo subscription-manager release --set=$REL_VER
•	sudo reposync -p $BASE_DIR --download-metadata --repo=rhel-8-for-x86_64-baseos-rpms
•	sudo reposync -p $BASE_DIR --download-metadata --repo=rhel-8-for-x86_64-appstream-rpms
3.	Update QA Symlink:
•	sudo ln -sfn $BASE_DIR /repo/rhel/$REL_VER/current

3. Phase 2: QA Validation
Performed on QA Client Servers.
1.	Execute Update:
sudo dnf clean all && sudo dnf update -y
2.	Capture Parity Audit:
find /repo/rhel/$REL_VER/snapshots/$SNAP_DATE -name "*.rpm" | wc -l > /repo/rhel/$REL_VER/snapshots/$SNAP_DATE/qa_count.txt
4. Phase 3: Tarball Bridge Migration
Securely move the data to the isolated Production NFS.
1.	Pack (QA Side):
cd /repo/rhel/$REL_VER/snapshots/
tar -czpf rhel_${REL_VER}_${SNAP_DATE}.tar.gz $SNAP_DATE/
sha256sum rhel_${REL_VER}_${SNAP_DATE}.tar.gz > rhel_${REL_VER}_${SNAP_DATE}.sha256
2.	Unpack (Prod Side): After transferring the files to the Production environment:
Bash
# Verify file integrity
sha256sum -c rhel_${REL_VER}_${SNAP_DATE}.sha256

# Extract to Prod NFS
sudo tar -xzf rhel_${REL_VER}_${SNAP_DATE}.tar.gz -C /repo/rhel/$REL_VER/snapshots/
3.	Promote (Update Prod Symlink):
Bash
sudo ln -sfn /repo/rhel/$REL_VER/snapshots/$SNAP_DATE /repo/rhel/$REL_VER/current
________________________________________
5. Phase 4: Production Execution
Performed on Production Client Servers.
1.	Verification Check:
Bash
# This count must match the number inside qa_count.txt
find /repo/rhel/$REL_VER/snapshots/$SNAP_DATE -name "*.rpm" | wc -l
2.	Execute Patching:
Bash
sudo dnf clean all && sudo dnf update -y
________________________________________
6. Maintenance & Retention
•	Cleanup: Delete .tar.gz and .sha256 files immediately after extraction to save space.
•	Snapshot Retention: Run this monthly to keep a rolling 90-day history.
Bash
find /repo/rhel/8.10/snapshots/ -maxdepth 1 -type d -mtime +90 -exec rm -rf {} +

