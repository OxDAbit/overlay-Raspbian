	/src
	├── overlay
	│   └── overlayRoot.sh 						# Original script v1.1 Created by Pascal Suter
	│
	└── overlay_sfs								# Custom resources
	    ├── boot 								# Contain boot files
	 	│	├── cmdline-no_overlay.txt 			# Boot Raspberry without OverlayFS
	 	│	└── cmdline-overlay.txt 			# Boot Raspberry with OverlayFS
	    └── sbin 								# Script to create OverlayFS and SquashFS
			└── overlayRoot.sh 					# SquashFS over OverlayFS script v1.4