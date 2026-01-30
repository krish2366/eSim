# List of Issues Encountered While Installing eSim on Ubuntu 25.04

This document describes the issues encountered while installing eSim on Ubuntu 25.04, listed in the order in which they were faced.
Each issue also includes its difficulty level and impact.

Ubuntu 25.04 was installed using Oracle VM on an existing Windows system. This setup does not differ functionally from a standard Ubuntu installation for the purposes of eSim installation.

## Issue 1 – eSim installer lacked support for Ubuntu 25.04

Difficulty: Easy
Impact: High (installation blocked)

The existing eSim installer script did not include explicit support for Ubuntu 25.04. When executed on this version, the installer failed because the script did not recognize or handle the new release, even though the system environment itself was compatible. As a result, users running Ubuntu 25.04 were completely unable to install eSim using the official installer, making this a high-impact issue.

Fix for Issue 1 – Reusing Ubuntu 24.04 installer logic for Ubuntu 25.04

To quickly enable compatibility with Ubuntu 25.04, the installer was modified to treat Ubuntu 25.04 the same as Ubuntu 24.04. This involved reusing the existing and well-tested installation logic for Ubuntu 24.04 whenever the script detects Ubuntu 25.04. Although this change was simple to implement, it had a high impact because it immediately allowed eSim to be installed and used on Ubuntu 25.04 without further modifications.

## Issue 2 – KiCad installation step failing or behaving inconsistently

Difficulty: Medium–High
Impact: High (affects GUI and PCB workflow)

The eSim installer is responsible for installing KiCad, which is essential for schematic capture and PCB design within eSim. On the tested Ubuntu setup, the KiCad installation step was unreliable: the package names or commands used in the script no longer aligned with the packages available on newer Ubuntu releases. As a result, KiCad was either not installed at all or an unusable version was installed.

Even when eSim itself was successfully installed, users could not use the GUI or PCB design workflow without manually fixing the KiCad installation. Since KiCad is a core dependency of eSim, this issue had a high impact on usability and a medium–high difficulty due to the need to understand distribution-specific package availability.

Fix for Issue 2 – Installing KiCad via Flatpak instead of apt

To resolve this, the KiCad installation method was changed from apt to Flatpak. The apt-based approach attempted to install a KiCad build that depended on libgit2-1.8, which was not available (or not available in the expected version) on the tested Ubuntu release, leading to installation or runtime failures.

By using Flatpak, KiCad is installed as a self-contained package with all required dependencies bundled. This removes reliance on the system’s libgit2-1.8 package and significantly improves reliability across supported Ubuntu versions. As a result, eSim’s GUI and PCB workflows function correctly out-of-the-box.

## Issue 3 – Failure during KiCad library copy step

Difficulty: Medium
Impact: Medium (affects KiCad integration)

The installer includes a step to copy KiCad libraries into the eSim environment so that symbols, footprints, and example projects are available. On the tested system, this step failed because the script assumed fixed KiCad installation paths and pre-existing destination directories.

When these assumptions did not match the actual setup, the copy commands either failed with errors or silently skipped files. This left the installation partially configured: KiCad was present, but important libraries required by eSim were missing, which degraded the overall user experience.

Fix for Issue 3 – Correcting the KiCad library copy logic

To address this, the installer was updated to remove reliance on hard-coded paths and existing directories. The source paths were aligned with the actual KiCad installation location used in the updated setup. Additionally, the installer now explicitly creates all required destination directories before copying files.

With these changes, KiCad libraries and related resources are consistently copied to the correct locations, ensuring that symbols, footprints, and example designs work as expected after installation.

## Issue 4 – GTK3-related failure during installation

Difficulty: Medium
Impact: Medium–High (affects GUI components)

Another issue encountered was related to GTK3 support. A GTK3-related check or configuration step in the installer was failing on the target Ubuntu system, despite the system being capable of running GTK3 applications. Because this check was treated as mandatory, the failure caused the installer to stop configuring GUI components instead of attempting to resolve the issue or proceeding safely.

As a result, the eSim graphical interface could not be used reliably without manual intervention, significantly impacting the out-of-the-box experience.

Fix for Issue 4 – Adjusting GTK3 handling in the installer

To resolve this, the installer was modified so that the GTK3 check no longer blocks an otherwise valid installation. Instead, the script now ensures that the required GTK3 packages or settings are present and then continues with the installation. Where necessary, the check was relaxed or corrected so that systems with functional GTK3 support are not incorrectly flagged as failures. This change allows eSim’s GUI components to install and function reliably across supported Ubuntu versions.

## Issue 5 – NGHDL installer did not recognize Ubuntu 25.04

Difficulty: Easy
Impact: High (blocks mixed-signal simulation)

The NGHDL installation script provided with eSim did not include support for Ubuntu 25.04. It only recognized older Ubuntu versions, so when run on Ubuntu 25.04, it failed to select a valid installation path and did not install NGHDL. Since NGHDL is required for mixed-signal simulations in eSim, this completely prevented users on Ubuntu 25.04 from using that functionality.

Fix for Issue 5 – Reusing Ubuntu 24.04 logic for NGHDL

This was fixed by extending the NGHDL installer to treat Ubuntu 25.04 in the same way as Ubuntu 24.04. The script now detects Ubuntu 25.04 and applies the same installation steps and package configuration known to work on Ubuntu 24.04. Although the change is simple, it immediately restores NGHDL support and enables mixed-signal simulations on Ubuntu 25.04.

## Issue 6 – NGHDL / GHDL LLVM backend build failure with newer LLVM versions

Difficulty: High
Impact: High (core simulation functionality broken)

On newer Ubuntu releases, the NGHDL installation failed while building the GHDL LLVM backend because the system uses LLVM 20 (llvm-config-20). The installer script assumed the presence of LLVM 15, specifically expecting llvm-config-15 at /usr/bin/llvm-config-15.

When this binary was not available—either because a different LLVM version was installed or LLVM 15 was not present—the ./configure step for GHDL failed, causing the entire NGHDL installation to stop. This issue is high impact because it breaks mixed-signal simulation, and high difficulty due to the need to manage LLVM version compatibility and GHDL build requirements.

Fix for Issue 6 – Enforcing LLVM 15 for stability

To fix this, the installation script was modified to explicitly check for llvm-config-15. If it was not found, LLVM 15 was installed and the script was updated to point to its correct path during the NGHDL build process (install-nghdl-24.04.sh). This approach ensures a stable and compatible LLVM version for building GHDL. The same mechanism can later be adapted to newer LLVM versions once they are confirmed to be compatible with GHDL.

## Summary

All of the above issues were identified and resolved using Ubuntu 25.04, Bash scripting, and relevant system resources. With these fixes applied, eSim was successfully installed and used on Ubuntu 25.04.

Some changes related to NGHDL scripts cannot be detailed here, as those scripts are part of external dependencies and are not included in the forked repository.

