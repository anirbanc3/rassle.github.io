## This is the website for RASSLE attack

For more details on the paper, you can follow this link [TCHES website] <https://tches.iacr.org/index.php/TCHES/article/view/8795/8395>


### Short Summary of the attack

In this attack, we show that the OpenSSL implementation of scalar multiplication on P-256 curve has varying number of add-and-sub function calls, which depends on the secret scalar bits. We demonstrate through several experiments that the number of add-and-sub function calls can be used to template the secret bit, which can be picked up by the spy using the principles of RASSLE. We eventually demonstrate a full end-to-end attack on OpenSSL ECDSA using curve parameters of curve P-256.  We particularly abuse the deadline scheduler policy to attain perfect synchronization between the spy and victim, without any aid of induced synchronization from the victim process. This synchronization and timing leakage through RASSLE  is sufficient to retrieve theMost Significant Bits (MSB) of the ephemeral nonces used while signature generation, from which we subsequently retrieve the secret signing key of the sender applying the Hidden Number Problem.


### How to recreate the attack?
#### System Specifications

The attack utilizes the deadline scheduler to achieve synchronization between the spy and victim. The demo have been tested on a system which has the following specifications:
- Processor -  Intel Xeon CPU E5-2609 v4 (Broadwell)
- Operating System - Red Hat Enterprise Linux Server 7.7 (kernel 3.10.0-1062.9.1.el7.x86_64)
- Deadline scheduler parameters – 
  - sched-runtime -> 3600
  - sched-deadline -> 3700
  - sched-period -> 7200
- Return Address Stack (RAS) size - 16
- OpenSSL version – 1.1.1g
- linux-util version - 2.31 

##### Pre-requisites

###### Setting up OpenSSL
The demo requires OpenSSL library to be installed in the system. A sample OpenSSL repository has been included with this repo. 
In order to install the specific version of OpenSSL -  
1. Untar or extract `openssl-1.1.1g-RASSLE.tar.xz`. 
2. `cd` into the extracted directory and configure it using `./config`.
3. Once the library has been configured for the system, execute `make` to build all the necessary files.
4. After the build is complete, complete the installation by executing `sudo make install`. This version of OpenSSL will be installed in `/usr/local/lib`.
5. [optional] Run `sudo ldconfig` in case the installation is not successful after the end of previous step (step 4).



### Setting up the deadline scheduler

Most Linux-based operating systems offer a number of scheduling policies, which are crucial artifacts for controlling two asynchronous processes' execution. Among the available policies, the deadline scheduler is particularly interesting because it imposes a "deadline" on operations to prevent starvation of processes. In the deadline scheduler, each request by a process to access a system resource has an expiration time. A process holding a system resource does not need to be forcefully preempted, as the deadline scheduler automatically preempts it from the CPU after its request expiration time.

In order to inspect the underline scheduler in the working system, run `cat /sys/block/sda/queue/scheduler`. The attack will work only if deadline scheduler is present (it does not have to be the default scheduler).

The operation of deadline scheduler depends on three parameters, namely 'runtime', 'period', and 'deadline'. These parameters can be adjusted using `chrt` command, which can be executed from user-level privilege by acquiring `CAP_SYS_NICE` permission. 

To acquire the required privilege, execute the following command `sudo setcap cap_sys_nice+ep /usr/bin/chrt`.

The command to run an `<executable>` using deadline scheduler is as follows:
  `chrt -d --sched-runtime t1 --sched-deadline t2 --sched-period t3 0 <executable>`
For this work, we set t1 with the approximate time (in nanoseconds) required to execute one iteration of ECC Montgomery ladder. Further, we set the parameter sched-deadline to a value t2 = t1 + δ such that the ECC process leaves the CPU after execution of a single Montgomery ladder iteration. We set the parameter sched-period to a value t3 = 2 × t1. 

## How to run the demo

The attack works in two phases - template building and template matching

### Template Building

1. Open a terminal. Run the shell script `script_template.sh` to build the templates. In our experiments, we consider the most significant bit (msb) to be 1 and build templates for 6 msbs. Therefore total number of templates built is 32 (keeping msb as 1). The script executes a spy process (which measures timing leakage using RASSLE) and a victim process (which is performing EC scalar multiplication) in a completely asynchronous setup under the influence of deadline-scheduler. The timing information observed by the spy is logged into text files by the name `filename_<nonce>_<count>.txt` where nonce varies from `100000` to `111111` (in decimal) and count varies from 1 to 10,000.
2. Once the above script ends, run `generate_template.py` to create the template dataset. The python script reads the timing files created in the earlier step and processes them further to create the dataset. The dataset is saved in the root folder by the name `rassle_timing_dataset.npy` as a numpy array.
3. Delete the file named `file_mont_ladder.txt` present in the same folder using the command `rm file_mont_ladder.txt`. This step is important for the attack to be successful.

### Template Matching

1. Open `ecc_encrypt.c` in a text editor. Comment out line 66 and save the file. During template building phase, we varied the 6 msbs while keeping the remaining bits same. But, during the matching phase, the nonces are generated at random. So all the 256 bits of the nonce are used as input in this case.
2. Open a terminal. Run the shell script `script_nonce.sh` to generate the datasets containing timing values obtained through RASSLE (similar to the template building phase). The script reads from a file containing random nonces and performs EC scalar multiplication using those nonces. The timing information observed by the spy is logged into text files by the name `filename_tst_<count>.txt` where count represents number of nonces used.
3. Once the above script ends, run `template_matching.py` to retrieve the candidate "partial nonces" using Least Square Error (LSQ) method. The python script reads the numpy array created during the building phase and compute medians for each bit of nonce in the timing dataset. These medians will act as representative template for a particular bit position of a particular sequence. We select the top 5 partial-nonce choices based on the LSQ scores and export them as csv file by the name `nonce_bits.csv`. The python script 
will also print the number of nonces correctly predicted.
4. Using these "partial nonces", the original secret signing key can be revealed using the well-known Lattice Attack.

