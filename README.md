# rclone-RAM-user
Script Bash per montare un remote `rclone` come filesystem locale tramite `rclone mount`, usando: una **directory locale visibile nella home** come mountpoint; una **directory nascosta nella home** come VFS cache; un `tmpfs` montato sulla directory cache, così la cache vive interamente in RAM.
