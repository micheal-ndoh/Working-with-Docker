# Working with Docker and User Permissions
In this exercise, you'll create and manage user permissions inside a Docker container using the BusyBox image. This includes creating users, managing groups, setting directory and file permissions, and persisting data across container runs.

## Step-by-Step Guide
 ### Step 1:
 Pull the BusyBox Image and Create a Container
First, we will pull the BusyBox image and launch a container in interactive mode.

```bash
docker pull busybox
docker run -it --name my-busybox-container busybox
```
### Step 2: 
Create Two Users and Add Them to a Group
Inside the container, follow these steps to create two users and assign them to a specific group:

Create the Group:

```
addgroup mygroup
```
Create Two Users and Add Them to the Group: Create two users (user1 and user2) and assign them to the mygroup group:

```
adduser -D -G mygroup user1
adduser -D -G mygroup user2
```
This creates users with default home directories and adds them to mygroup.

### Step 3: 
Create a Directory with Group Ownership and Permissions
Next, we create a shared directory and set the correct permissions so that both users can access and modify files:

Create the Directory:

```
mkdir /shared-dir
```
Change the Ownership of the Directory: Set the owner of the directory to be user1 and the group to mygroup:

```
chown user1:mygroup /shared-dir
```
Set Group Permissions: Give the group read, write, and execute permissions:

```
chmod 770 /shared-dir
```
Ensure Files Created in the Directory Inherit Group Ownership: Use the setgid bit so that any files created inside the directory are automatically owned by mygroup:

```
chmod g+s /shared-dir
```
### Step 4: 
Restrict Deletion Permissions
To ensure only the file's owner can delete files, but the group can still access and modify them, follow these steps:

Use the Sticky Bit: The sticky bit restricts the ability to delete files in the directory to only the file's owner (or the root user), even though other users in the group can modify the files:
```
chmod +t /shared-dir
```
This ensures that only the owner of a file can delete it, even though all group members have read and write permissions.

### Step 5:
Persist Data Across Container Runs
To persist data across container runs, you'll use Docker volumes to store the shared directory's data on the host system. This way, the data will remain even if the container is stopped or removed.

Exit the Container:

```
exit
```
Create a Docker Volume: Create a named Docker volume to store the shared directory:
```
docker volume create mydata
```
Start the Container with the Volume Mounted: Run the container with the volume mounted to /shared-dir:


```
docker run -it --name my-busybox-container -v mydata:/shared-dir busybox
```
Any changes made to the /shared-dir directory will now persist across container restarts.

### Step 6: 
Verify Permissions and Functionality
Now, let's test the functionality to ensure everything is set up correctly.

Switch to User1 and create a file in /shared-dir:

```
su user1
touch /shared-dir/file1
```
Switch to User2 and attempt to modify the file but not delete it:

```
su user2
echo "User2 modifies this file" >> /shared-dir/file1
rm /shared-dir/file1  # This should fail since user2 isn't the owner.
```
Check that the file is still there and verify the file content:
```
cat /shared-dir/file1
```
Restart the Container and verify that the data in /shared-dir is persisted:

```
docker stop my-busybox-container
```

```
docker start my-busybox-container
```

```
docker exec -it my-busybox-container /bin/sh
```

```
ls /shared-dir
```
