#### Group ID: 15
#### Group Members: Aditi Raj Kandoi, Swetang Finviya, Yash Jetwani

We have also submitted a separate detailed design doc as: CSE-506/desgin.pdf
</br> and a complete flowchart design of our kernel APIs as: Kernel_API_Flowchart.pdf


-----

Our submission contains following files and folders:

1. Makefile
2. install_module.sh
3. kernel.config
4. async_ops_module.c
5. async_ops_module.h
6. async_ops.c
7. async_ops.h
8. test-scripts
9. run_all_tests.sh

----- 

**Makefile:**
	Contains commands to build and compile module as well as user level code.

**install_module.sh:**
    Shell script to insert newly compiled async_ops_module.ko module and remove previously installed version of the module, if any.

**kernel.config:**
	Contains kernel configuration used for booting the kernel (We need some compression and crypto modules over vanilla which are included in this kernel config, should be used as Linux's .config while making kernel)

**async_ops_module.c:**
	Contains kernel side workqueue APIs and operation code

**async_ops_module.h:**
	Header file for async_ops_module.c

**async_ops.c:**
    Contains user level code for handling command line arguments and communicating with the kernel module async_ops_module

**async_ops.h:**
	Header file for async_ops.c, shared by kernel module to refer input and return structs

**test_scripts:**
	Folder containing all the unit and integrating tests

**run_all_tests:**
	Bash script to run all the test scripts present in test_scripts folder

----

**Steps to build and compile async_ops_module(kernel module) and async_ops(user level code)**<br>
Go to the folder /usr/src/hw3-cse506g15/CSE-506<br>
Run the following command: sudo make<br>
Run the following command: sudo sh install_module.sh<br>

Job types supported by the work queue:
- Delete multiple files
- Rename multiple files
- Concatenate multiple files
- stat() multiple files
- Calculate hash of a file
- Encrypt or decrypt a file
- Compress or decompress a file

Job types usage and descriptionâ€”

1. Delete multiple files
	
	Usage: ./async_ops delete -n N -i file1 file2.. fileN
	
	Flags:
		-n: number of files
		-i: the files to be deleted

	This command will delete all the files provided by the user or returns an appropriate error.

2. Rename multiple files
	
	Usage: ./async_ops rename -n N -i file1 file2.. fileN -o output1 output2.. outputN

	Flags:
		-n: number of files
		-i: the files to be renamed
		-o: new names of the files

This command will rename the files to the file names given by the user after the -o flag.

3. Concatenate multiple files

	Usage: ./async_ops concatenate -n N -i file1 file2.. fileN -o output
	Flags:
		-n: number of files
		-i: the files to be renamed
		-o: the concatenated file

This command will concatenate the files and store the content in the output file. If any of the input files does not exist, the operation fails and no output file is created.

4. Getting the stat of  multiple files
	
	Usage: ./async_ops stat -n N -i file1 file2.. fileN -o output1 output2.. outputN

	Flags:
		-n: number of files
		-i: the files to be renamed
		-o: output files which will store the stat

This command will generate the stat for all  the files provided and store their stat to their respective output files.

5. Calculate hash of a file

Usage: ./async_ops hash file.txt result.txt

This command will calculate the hash of file.txt and store it in result.txt. md5 algorithm is used for computing the hash. The value of the hash stored in result.txt will be the same as result of following command: md5sum file.txt.

6. Encrypt or decrypt a file   

Usage: ./async_ops crypt {-e|-d} -p â€śpassword phraseâ€ť file1.txt file2.txt

Flags:
	-e: for encryption
	-d: for decryption
	-p: for password

This command will encrypt/decrypt the text present in file1.txt and store the encrypted/decrypted text in file2.txt. The key used for encryption/decryption is derived from the password phrase.

For decryption, if the wrong key is used then the error -EKEYREJECTED will be reported

7. Compress/decompress a file

	Usage: ./async_ops comp {-c|-d} file1.txt file2.txt

	Flags:
		-c: for compression
		-d: for decryption

This command will compress/decompress the content present in file1.txt and store the compressed/decompressed  content in file2.txt. lz4 algorithm is used for this purpose.

-----

**Kernel Queue APIs supported** 

1. Submitting a job

    Submitting a job is performed by the APIs above. Each correct operation command adds that operation with the passed parameters to the queue.

2. List all pending/running jobs in the queue [We have limits to the number of jobs in active queue, can be tweaked using MAX_ACTIVE_JOBS macro]

    Usage: ./async_ops list_jobs

    Sample output:

    JOB_ID          OPERATION           STATUS          PRIORITY            TIMEonQ(s)          USER_ID
    1               hash                RUNNING         normal              4                   0
    2               compress            PENDING         high                16                  1000

    All signifiers carry their usual meaning. Notes on some of them â€”
        STATUS: Status can only be PENDING/RUNNING cause this API list jobs from the active queue, completed jobs are kept in a separate list and can be listed using a separate API.

        TIMEonQ(s): Time elapsed in seconds since the job has been submitted to the queue

3. List all completed jobs [We have limits to the number of jobs in completed queue, can be tweaked using MAX_COMPLETED_JOBS_COUNT macro]

    Usage: ./async_jobs get_completed_jobs

    Output: Returns a list of jobs that have completed execution. Their results can be subsequently polled using a separate API.

4. Get status/poll result of a job

    Usage: ./async_jobs get_status job_id

    Sample Output 1:

        Job ID: 12 - RUNNING

    Sample Output 2:

        Job ID: 10 - PENDING

    Sample Output 3:

        Job ID:12 - Completed
        Operation status: -2

    Sample Output 4:

        Job ID: 12 â€” Completed
        /usr/src/hw3-cse506g15/CSE-506/12.out: 0
        /usr/src/hw3-cse506g15/CSE-506/13.out: 0
        /usr/src/hw3-cse506g15/CSE-506/14.out: -2

    All signifiers carry their usual meaning. Notes on some of them â€”
    Sample output 4 returns filewise operation status. This can occue in case of operations on multiple files, like deleting, stat, renaming multiple files in one job.

5. Cancel/Remove job from queue

    Usage: ./async delete_job job_id

    Return: Error if delete fails

    This API can be used to remove one job from the queue at a time. Note that the operation suceeds only if the job is in pending state.

6. Boost priority of a job

    Usage: ./async priority_boost job_id

    Return: Error if priority boost fails

    This API can be used to boost priority of one job at a time. The job is deallocated from the "normal" queue and allocated to the "high" priority queue. Note that the operation suceeds only if the job is in pending state.



ALL the above APIs have user id filtering, ie. only jobs owned by user are visible to them in return responses of all APIs.

-----

**Future Developments**

1. We plan to introduce submitting jobs using shell script (Slurm standard). This method could be used to automate job submission and timer activities.
2. We plan to provide a time analytics API which will contain lifetime information of a job â€” Time spent in pending state on queue, time on CPU, total time it took to reach COMPLETED state etc. This API can be used to increase system efficiency.

----

**References**

https://kernelnewbies.org/FAQ/LinkedLists

https://www.oreilly.com/library/view/linux-device-drivers/0596005903/ch07.html

https://www.oreilly.com/library/view/understanding-the-linux/0596005652/ch04s08.html

https://elixir.bootlin.com/linux/latest/source/arch/s390/crypto/arch_random.c#L104

https://lwn.net/Articles/403891/

https://linuxtv.org/downloads/v4l-dvb-internals/device-drivers/ch01s06.html

https://docs.google.com/document/d/1mMS2EBXc_LJvdEcHo5baruvx2LRNE9P5_t3ZQHoaErM/edit


$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
