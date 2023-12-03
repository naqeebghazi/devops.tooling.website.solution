# Partitions on Block Devices on Linux

## Creating Partitions via  terminal:

Creating partitions on an AWS EC2 instance involves several steps, including identifying the available block devices, using partitioning tools, and creating partitions of different sizes. Below is a general guide on how to create partitions with different sizes on an AWS EC2 instance:

Important Note:
Before proceeding, make sure to back up any important data on your instance. Creating partitions involves modifying the disk structure, and there is a risk of data loss if not done carefully.

### Steps to Create Partitions:
  
1. Identify Block Devices:
  
Connect to your EC2 instance using SSH.
  
Use the lsblk command to identify the block devices available on your instance.
  
    lsblk
  
Identify the block device you want to partition (e.g., /dev/xvda).

The mount command tells us what storage devices are mounted on our system:

    mount | grep xvdb
    
We can use grep to narrow down the output for our devices

2. Partition the Block Device:

To completely wipe the disk and create a band new partition table on it:
First unmount the disk

    sudo umount /dev/xvdb  # umount is not a typo
    mount | grep xvdb      # this to double check the disk is no longer listed and therefore has been unmounted
    
Use a partitioning tool like fdisk to create partitions on the chosen block device.

    sudo fdisk /dev/xvda

Follow the prompts to create partitions. You can create multiple partitions with different sizes.

  p - prints current partition table
  m - helpful cmd list
  g - creates a gpt type partition table (default)
  n - new partition table process
  

3. Format Partitions:

After creating partitions, use the mkfs command to format each partition with a filesystem.

    sudo mkfs -t ext4 /dev/xvda1   # Replace /dev/xvda1 with the actual partition
    sudo mkfs -t ext4 /dev/xvda2   # Replace /dev/xvda2 with another partition, if created

Repeat this step for each partition.

4. Create Mount Points:

Create directories to serve as mount points for each partition.

    sudo mkdir /mnt/partition1
    sudo mkdir /mnt/partition2

Adjust the directory names based on your preferences.

5. Mount Partitions:

Mount each partition to its corresponding mount point.

    sudo mount /dev/xvda1 /mnt/partition1
    sudo mount /dev/xvda2 /mnt/partition2

Adjust the device names and mount points accordingly.

6. Update /etc/fstab for Persistence:

Add entries to the /etc/fstab file to ensure that the partitions are mounted automatically at boot.

    echo "/dev/xvda1 /mnt/partition1 ext4 defaults 0 0" | sudo tee -a /etc/fstab
    echo "/dev/xvda2 /mnt/partition2 ext4 defaults 0 0" | sudo tee -a /etc/fstab

Adjust the device names, mount points, and filesystem types based on your configuration.

7. Verify Configuration:

Check the mounted partitions and their sizes.

    df -h

Verify that each partition is correctly mounted.

By following these steps, you can create partitions of different sizes on your AWS EC2 instance. Adapt the commands and partition sizes based on your specific requirements.


## Deleting Partitions:

If you've created partitions using gdisk and want to remove them, you'll need to use the same tool to delete the partitions. Here's a step-by-step guide:

Using gdisk to Remove Partitions:
1. Launch gdisk on the Block Device:

  Replace /dev/xvda with your actual block device.
  
      sudo gdisk /dev/xvda

2. View the Current Partition Table:

  Type p and press Enter to print the current partition table.

  Note the partition numbers you want to delete.

3. Delete Partitions:

  Type d and press Enter to delete a partition.
  
  Enter the partition number when prompted.

4. Repeat for Additional Partitions:

  If you have multiple partitions to delete, repeat the d command for each one.

5. Command (? for help):

  After deleting partitions, type p to print the partition table and verify the changes.

6. Write Changes to Disk:

  If you are satisfied with the changes, type w and press Enter to write the changes to the disk.
  Confirm the action by typing Y and pressing Enter.

Quit:

  Type q and press Enter to quit gdisk.

Important Notes:
  - Deleting partitions using gdisk removes the partition entries but doesn't immediately erase the data within the deleted partitions.
  - If you want to reclaim space and remove filesystems within those partitions, you'll need to use tools like mkfs to delete the filesystems first.
  - Deleting partitions may result in data loss, so ensure that you have backed up any important data before proceeding.

After completing these steps, the specified partitions should be deleted, and the changes will be written to the disk. Always exercise caution when modifying disk partitions, as it can lead to data loss if not done carefully.
