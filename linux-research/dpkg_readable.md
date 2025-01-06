Dpkg
Debian based package manager older than new tool apt (aptitude) which lets you install, remove and mange linux packages

structure: 
>>> dpkg [option] [.deb package name]

some of important commands uses:
>>> dpkg -i .deb_package	// installing a package
>>> dpkg -r .deb_package	// removing a package but without config files
>>> dpkg -P .deb_package	// removing a package completely
>>> dpkg –-update-avail		// update repo

(tip) search a installed package name :
>>> dpkg -l | grep “name_you_want_to_search”
