Download Link: https://assignmentchef.com/product/solved-ecse427-assignment-3-mountable-simple-file-system
<br>



<ol>

 <li><strong> What is required as part of this assignment? </strong></li>

</ol>

In this assignment, you are expected to design and implement a simple file system (SFS) that can be mounted by the user under a directory in the user’s machine. <strong>You need to demonstrate the SFS working only in Linux.</strong> The SFS introduces many limitations such as restricted filename lengths, no user concept, no protection among files, no support for concurrent access, etc. You could introduce additional restrictions in your design. However, such restrictions <strong>should be reasonable to not oversimplify the implementation and should be documented in your submission</strong>. Even with the said restrictions, the file system you are implementing is highly useable for embedded applications. Here is a list of restrictions of the simple file system as specified in the handout:

<ul>

 <li>Limited length filenames (select an upper limit such as 16)</li>

 <li>Limited length file extensions (could be set to 3 – following the common extension length)</li>

 <li>No subdirectories (only a single root directory – this is a severe restriction – relaxing this would enable your file system to run many applications)</li>

 <li>Your file system is implemented over an emulated disk system, which is provided to you.</li>

</ul>

Here is a schematic that illustrates the overall concept of the mountable simple file system.

The gray colored modules in the above schematic are provided by the Linux OS. The blue colored modules are give to you as part of the support code provided as part of the assignment. You are expected to develop the yellow colored module.

<h1>2.      Objectives in detail</h1>

In reality, you could implement the SFS with <strong>your own API that implements the necessary functions to interface with the FUSE wrapper</strong> provided as part of this assignment. However, for debugging purposes, we suggest that you implement your file system such that it exposes the following API. The additional test suite we provide with the assignment could be used to test your file system if you stick to the proposed API. You can deviate even significantly from the proposed API; however, in that case you will be responsible for modifying the test suite.

The suggested API for SFS is given below. The API is based on C language. It is strongly suggested that you retain the functionality provided by the API if you decide to change it.




<table width="643">

 <tbody>

  <tr>

   <td width="305"><strong>void mksfs(int fresh);             </strong></td>

   <td width="337"><strong>// creates the file system </strong></td>

  </tr>

  <tr>

   <td width="305"><strong>int sfs_getnextfilename(char *fname);  </strong></td>

   <td width="337"><strong>// get the name of the next file in directory </strong></td>

  </tr>

  <tr>

   <td width="305"><strong>int sfs_getfilesize(const char* path); </strong></td>

   <td width="337"><strong>// get the size of the given file </strong></td>

  </tr>

  <tr>

   <td width="305"><strong>int sfs_fopen(char *name);         </strong></td>

   <td width="337"><strong>// opens the given file </strong></td>

  </tr>

  <tr>

   <td width="305"><strong>int sfs_fclose(int fileID);       int sfs_fwrite(int fileID,  </strong></td>

   <td width="337"><strong>// closes the given file </strong></td>

  </tr>

  <tr>

   <td width="305"><strong>         char *buf, int length);  </strong><strong>int sfs_fread(int fileID, </strong></td>

   <td width="337"><strong>// write buf characters into disk </strong></td>

  </tr>

  <tr>

   <td width="305"><strong>         char *buf, int length);  </strong><strong>int sfs_fseek(int fileID, </strong></td>

   <td width="337"><strong>// read characters from disk into buf </strong></td>

  </tr>

  <tr>

   <td width="305"><strong>         int loc);                  </strong></td>

   <td width="337"><strong>// seek to the location from beginning </strong></td>

  </tr>

  <tr>

   <td width="305"><strong>int sfs_remove(char *file);        </strong></td>

   <td width="337"><strong>// removes a file from the filesystem </strong></td>

  </tr>

 </tbody>

</table>

<strong> </strong>

The <strong>mksfs()</strong> formats the virtual disk implemented by the disk emulator and creates an instance of the simple file system on top of it. The <strong>mksfs()</strong> has a fresh flag to signal that the file system should be created from scratch. If flag is false, the file system is opened from the disk (i.e., we assume that a valid file system is already there in the file system. The support for persistence is important so you can reuse existing data or create a new file system.

The <strong>sfs_getnextfilename(char *fname)</strong> copies the name of the next file in the directory into fname and returns non zero if there is a new file. Once all the files have been returned, this function returns 0. So, you should be able to use this function to loop through the directory. In implementing this function, you need to ensure that the function remembers the current position in the directory at each call. Remember in SFS we have a single-level directory. The <strong>sfs_getfilesize(char *path)</strong> returns the size of a given file.

The <strong>sfs_fopen()</strong> opens a file and returns the index that corresponds to the newly opened file in the file descriptor table. If the file does not exist, it creates a new file and sets its size to 0. If the file exists, the file is opened in append mode (i.e., set the file pointer to the end of the file). The <strong>sfs_fclose()</strong> closes a file, i.e., removes the entry from the open file descriptor table. On success, <strong>sfs_fclose()</strong> should return 0 and a negative value otherwise. The <strong>sfs_fwrite()</strong> writes the given number of bytes of buffered data in <strong>buf</strong> into the open file, starting from the current file pointer. This in effect could increase the size of the file by the given number of bytes (it may not increase the file size by the number of bytes written if the write pointer is located at a location other than the end of the file). The <strong>sfs_fwrite()</strong> should return the number of bytes written. The <strong>sfs_fread()</strong> follows a similar behavior. The <strong>sfs_fseek()</strong> moves the read/write pointer (a single pointer in SFS) to the given location. It return 0 on success and a negative integer value otherwise. The <strong>sfs_remove()</strong> removes the file from the directory entry, releases the file allocation table entries and releases the data blocks used by the file, so that they can be used by new files in the future.

A file system is somewhat different from other components because it maintains data structures in memory as well as disk! The disk data structures are important to manage the space in disk and allocate and de-allocate the disk space in an intelligent manner. Also, the disk data structures indicate where a file is allocated. This information is necessary to access the file.

<h1>3.      Implementation strategy</h1>

The disk emulator given to you provides a constant-cost disk (CCdisk). This CCdisk can be considered as an array of sectors (blocks of fixed size). You can randomly access any given sector for reading or writing. The CCdisk is implemented as a file on the actual file system. Therefore, the data you store in the CCdisk is persistent across program invocations. To mimic the real disk, the CCdisk is divided into sectors of fixed size. For example, we can split the space into 1024 byte sectors. The number of sectors times the size of a sector gives the total size of the disk. In addition to holding the actual file and directory data, we need to store auxiliary data (meta data) that describes the files and directories in the disk. The structure and number of bytes spent on meta data storage depends on the file system design, which is the concern in this assignment.

On-disk data structures of the file system include a “super” block, the root directory, free block list, and i-Node table. The figure below shows a schematic of the on-disk organization of SFS.




<table width="455">

 <tbody>

  <tr>

   <td width="52">Super&amp; Block</td>

   <td width="138">I-node&amp;Table</td>

   <td width="192">Data&amp;Blocks</td>

   <td width="73">Free&amp;BitMap</td>

  </tr>

 </tbody>

</table>

The super block defines the file system geometry. It is also the first block in SFS. So the super block needs to have some form of identification to inform the program what type of file system format is followed for storing the data. The figure below shows the proposed structure for the super block. We expect your file system to implement these features, but some modifications are acceptable provided they are well documented. Each field in the figure is 4 bytes long. For instance, the magic number field is 4 bytes long. With a 1024-byte long block (recommended size), we can see that there will plenty of unused space in the super block.

<table width="145">

 <tbody>

  <tr>

   <td width="145">Magic       (0xACBD0005)</td>

  </tr>

  <tr>

   <td width="145">Block        Size          (1024)</td>

  </tr>

  <tr>

   <td width="145"></td>

  </tr>

  <tr>

   <td width="145">i-Node      Table        Length                (#             blks)</td>

  </tr>

  <tr>

   <td width="145">Root         Directory (i-Node                #)</td>

  </tr>

  <tr>

   <td width="145">Unused</td>

  </tr>

 </tbody>

</table>

A file or directory in SFS is defined by an i-Node. Remember we simplified the SFS by just having a single root directory (no subdirectories). This root directory is pointed to by an i-Node, which is pointed to by the super block. The i-Node structure we use here is slightly simplified too. It does not have the double and triple indirect pointers. It has direct and single indirect pointers. With the i-Node all the meta information (size, mode, ownership) can be associated with the i-Node. So, the directory entry can be pretty simple. The figure below shows the simplified i-Node structure.

<table width="326">

 <tbody>

  <tr>

   <td width="29"></td>

   <td width="29"></td>

   <td width="29"></td>

   <td width="29"></td>

   <td width="29"></td>

   <td width="29"></td>

   <td width="29"></td>

   <td width="67"></td>

   <td width="29"></td>

   <td width="29"></td>

  </tr>

 </tbody>

</table>

We are suggesting the i-Node structure shown above to maintain a semblance of similarity to the UNIX file system. However, the simplification made to the SFS i-Nodes already makes it impossible to read or write the SFS using UNIX software or vice-versa.

The directory is a mapping table to convert the file name to the i-Node. Remember a file name can have an extension too. You can limit the extension to 3 characters max. The file name (without extension) could be limited as well (16 characters is suggested). A directory entry is a structure that contains two fields (at least): i-Node and file name. You could add other fields (if you find necessary). Remember the i-Node also has some attributes such as mode, etc. Depending on the number of entries you have in the directory, the directory could be spanning across multiple blocks in the disk. The i-Node pointing to the root directory is stored in the super block so we know how to access the root directory. We assume that the SFS root directory would not grow larger than the max file size we could accommodate in SFS.

In addition to the on-disk data structures, we need a set of in-memory data structures to implement the file system. The in-memory data structures improve the performance of the file system by caching the on disk information in memory. Two data structures should be used in this assignment: directory table and iNode cache. The directory table keeps a copy of the directory block in memory. Don’t make the simplification of limiting the root directory to a single block (this would severely restrict the size of the disk – by limiting the number of files in disk). Instead, you could either read the whole directory into the memory or have a cache for the currently used directory block. The later one could be hard to get right.

Further, when you want to create, delete, read, or write a file, first operation is to find the appropriate directory entry. Therefore, directory table is a highly accessed data structure and is a good candidate to keep in memory. Another data structure to cache in the memory is the free block list. See the class notes for different implementation strategies for the free block list.

The figure below shows the in-memory data structure and how it connects to the other components. We need at least a table that combines the open file descriptor tables (the per-process one and system-wide one) in a UNIX-like operating system. We simplify the situation because we assume that only one process is accessing a file at any given time.

When a file is opened, we create an entry in this table. The index of the newly created entry is the “file descriptor” that is return by the file opening activity. That is the return value of the <strong>sfs_fopen()</strong> is this index. The entry created in the file descriptor table has at least two pieces of important information: i-Node number and a read/write pointer. The i-Node number is the one that is the one that corresponds to the file. Remember just like there is an i-Node for the root directory, there is one i-Node associated with each file. When a file is opened that i-Node is number is recorded in this table entry. The read/write pointer is also set according to the file system operating rule. For instance, in this assignment (SFS), you are going to set the read/write pointer to the end of the file at open so that data written into the file will be appended to the file. In SFS, <strong>sfs_fseek()</strong> is a direct way of setting the read/write pointer value. The interesting problem you could be faced with is what to do when you perform a read or write after setting the read/write pointer. Specifically, if we have a single pointer then <strong>sfs_fread()</strong> would also advance the “write” pointer and similarly <strong>sfs_fwrite()</strong> would advance the “read” pointer as well. We can simplify the complexity and let it be that way. You could opt to implement the SFS with two independent read and write pointers as well. In that case, the <strong>sfs_fseek()</strong> needs to have a parameter to specify whether the read, write, or both pointers should be moved by the seek operation.

As shown in the figure below, we have in-memory data structures and on-disk data structures in a file system. The in-memory data structures are activated as soon as the file system is up and running and they are updated every time a file system operation is carried out. While designing and implementing a given file system operation you need to think of the actions that should be carried out on the in-memory and ondisk data structures. In addition to the Open File Descriptor Table, we have variety of different caches for i-Nodes, disk blocks and the root directory. Your design could implement all of them or some of them. File system performance is not a concern for this assignment – correct operation is what we need.

<strong> </strong>

<strong><u>Rough pseudo code for creating a file:</u>  </strong>

<ol>

 <li>Allocate and initialize an i-Node. You need to somehow remember the state of the i-Node table to know which i-Node could be allocated for the newly created file. Simply remembering the last iNode used is not correct because as you delete files, some i-Nodes in the middle of the table will become unused and available for reuse.</li>

 <li>Write the mapping between the i-Node and file name in the root directory. You could simply update the memory and disk copies.</li>

 <li>No disk data block allocated. File size is set to 0.</li>

 <li>This can also “open” the file for transactions (read and write). Note that the SFS API does not have a separate create() call. So you can do this activity as part of the open() call.</li>

</ol>




<strong><u>Rough pseudo code for writing to a file:</u>  </strong>

<ol>

 <li>Allocate disk blocks (mark them as allocated in your free block list).</li>

 <li>Modify the file’s i-Node to point to these blocks.</li>

 <li>Write the data the user gives to these blocks.</li>

 <li>Flush all modifications to disk.</li>

 <li>Note that all writes to disk are at block sizes. If you are writing few bytes into a file, this might actually end up writing a block to next. So if you are writing to an existing file, it is important you read the last block and set the write pointer to the end of file. The bytes you want to write goes to the end of the previous bytes that are already part of the file. After you have written the bytes, you flush the block to the disk.</li>

</ol>




<strong><u>Rough pseudo code to seek on a file:</u>  </strong>

<ol>

 <li>Modify the read and write pointers in memory. There is nothing to be done on disk!</li>

</ol>







<h1>4.      More About Running the File System</h1>

We have given you a Makefile, disk emulator (C and Header), SFS test files, and FUSE wrappers. The Makefile given has five configurations. The first three use hand coded test files to test your implementation. Getting your implementing running with these three test files will get you a maximum of 95% grade. If you get it working with all five tests you can get a maximum of 110% (10% bonus). Note you should edit the EXECUTABLE value to reflect your name.

Also with the assignment package, we have given a working SFS file system that uses FUSE. You can use this file system in the Trottier Lab machines. Log into the <strong>labX-Y.cs.mcgill.ca (e.g., lab2-2)</strong> machines. Create a temporary directory in the folder that contains your SFS executable (e.g., <strong>mytemp</strong>). Now run the command

<strong>MyFilesystem_sfs mytemp</strong>

Run the <strong>ls</strong> command on <strong>mytemp</strong> and you will see nothing – it is an empty directory. That is the file system is empty. Now you copy some files over there or launch an editor like vi or emacs and create some files.

You will see <strong>fs.sfs</strong> file in the folder where you ran the <strong>MyFilesystem_sfs</strong> from. This file is your file system. The data in the files you copied or created are stored over here. To check the contents of the file, load it into an editor that will let you examine binary data (e.g., emacs). To un-mount the file system, find the process that is running the file system and kill it. Sometimes killing the process will leave the mount point corrupted (i.e., you cannot run the file system on the same mount point). Use the following command to clean up the corrupted mount point.

<strong>fusermount -u mytemp</strong>





