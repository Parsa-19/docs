# Dpkg
Debian based package manager older than new tool apt (aptitude) which lets you install, remove and mange linux packages.

structure: 
`dpkg [option] [.deb package name]`

### some of important commands uses:
isntalling a package: <br>
`dpkg -i .deb_package`

removing a package but without config files: <br>
`dpkg -r .deb_package` 

removing a package compeletely: <br>
`dpkg -P .deb_package` 

update repo: <br>
`dpkg –-update-avai` 

> [!tip]
> search a installed package name by `dpkg -l | grep "name_you_want_to_search"`
