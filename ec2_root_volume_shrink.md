# Decreasing Root Volume Size - EC2 Instance

Requirement:

 * One EC2 Machine for transferring data. Name it as copy_server
 * Snapshot of ec2 volume which we want to shrink. (Say 150GB - root volume)

Processor:

1. Create a new ec2 instance with the volume size you wanted to shrink existing root volume.
2. Detach the volumes you wish to resize from the current EC2 instance and from the newly created EC2 instance and attach them to the **copy server.**
3. Mount the old Volume as **/dev/sdf** which becomes **/dev/xvdf**
4. Mount the new Volume as **/dev/sdg** which becomes **/dev/xvdg**


  ```bash
  mkdir -p /vol/old
  mount -t ext4 /dev/xvdf /vol/old
  mkdir -p /vol/new
  mount -t ext4 /dev/xvdg /vol/new
  ```

5. Use rsync to copy the entire data from old vol to new vol

  ```bash
  rsync -az /vol/old /vol/new
  ```

6. Check UUID are same in both the volumes. (Ideally should be same in both the volumes)

  ```bash
  grep -nR "UUID" /vol/new/boot/grub/grub.cfg
  grep -nR "UUID" /vol/old/boot/grub/grub.cfg
  ```
  
  _Result from the above two commands after **UUID=** should have the same number._

7. Check both the volumes should have the same e2label.
8. If e2labels are not same. Change the e2lable of new volume as old volume.

  ```bash
  e2label /dev/xvdf1
  #cloudimg-rootfs
  e2label /dev/xvdg1
  # IF THIS RETURNS OTHER THAN THE ABOVE RESULT THEN RUN THE FOLLOWING
  e2label /dev/xvdg1 `e2label /dev/xvdf1`
  ```
  
9. Remove the volumes from the copy_server.

  ```bash
  umount /vol/new
  umount /vol/old
  ```

10. Attach the volume to new system as **/dev/sda1**. Boot it. 

*Instead of creating new EC2 Instance for new volume, you can directly attach new volume you wanted. But before proceeding any other steps mentioned here, you actully have to setup the partitions and boot flag. Else it will continue to boot as there is won't be any boot flag. You can follow [this](http://www.howtogeek.com/106873/how-to-use-fdisk-to-manage-partitions-on-linux/) to set up the boot flag on bare EC2 volume.*

Tested it in Druid Production setup to reduce all the servers 150GB Root Volume to 20Gb. Working. 
