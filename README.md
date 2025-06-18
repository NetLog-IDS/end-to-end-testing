# End to End Testing Repository
This repository is for guide on how to do end to end testing for all IDS systems, collecting test results, and processing it.

## IDS Systems
There are four IDS systems that you can test. Those systems' name are:
1. `both-old`: Netlog old and rule-based KSQL.
2. `netlog-new-only`: Netlog new (improved Netlog old) and rule-based KSQL
3. `ml-only`: Netlog old and machine learning.
4. `both-new`: Netlog new and machine learning

For more details, see the ```infrastructure``` repository on [this link](https://github.com/NetLog-IDS/infrastructure).

## How to run the test and get the data:
0. Prerequisite:
    - Clone the ```infrastructure``` repository on [this link](https://github.com/NetLog-IDS/infrastructure). 
    - Download the test file from [this link](https://mega.nz/folder/6yhAFLKJ#ElywB5FFqoKm-XpRXQ1ofw) and stores it on Google Drive.
    - Set up AWS CLI on the folder resulted in cloning the `infrastructure` repository.
1. Go to `/terraform` directory on `infrastructure` folder after cloning the repository. Verify if you can access AWS CLI first by tunning `aws sts get-caller-identity`. After that, run `terraform plan -var-file="./variables/general.tfvars.json" -var-file="./variables/specs/system-name.tfvars.json"`. Change system-name to the IDS system you want to test.
2. Run `terraform apply -var-file="./variables/general.tfvars.json" -var-file="./variables/specs/system-name.tfvars.json"`. Change system-name to the system you want to test.
3. For ml-only and both-new system, set up Apache Flink for the ML. Here's how:
   - Go to http://[ML_IP]:8082. You should see the Apache Flink page.
   - Go to 'Submit New Job' and then clink 'Add new'. Choose the flinkflowmeter-0.X.Y.jar files.
   - After the .jar files uploaded successfully, submit new job by clicking on the file name then on the 'Program Arguments' put `--source kafka --source-servers [KAFKA_PRIVATE_IP]:19092 --source-topic network-traffic --source-group flink12 --sink kafka --sink-servers [KAFKA_PRIVATE_IP]:19092 --sink-topic network-flows --mode ordered --sink-topic network-flows --mode ordered`.
   - Go to 'Running Job' page to confirm that the job has been run.
4. Run manual scripts for installing dependecies and downloading test file on Netlog VM. **Make sure to change the file id that you downloaded using gdown with the your own test file id!**. You can find the id on your file's shareable google drive link in format `https://drive.google.com/file/d/<file-id>`. Don't forget to share the files with access settings "Anyone with the link can view".
5. Open IDS dashboard on http://[MONITORING_IP]:8000 in your prefered browser. **Make sure that it doesn't get closed during the whole testing process!**
6. Run Netlog container using the command provided in `/manual/netlog-old.sh` or `/manual/netlog-new.sh` depending on what kinds of IDS system you want to test. **Make sure the terminal that runs the command doesn't get closed during the whole testing process!**
7. Run `tcpreplay` command with the .pcap files you want to test. Run the command **in a different terminal** with the command for the Netlog container. The command is on the same Netlog manual script files. Confirm that the IDS dashboard starts to display data. **Make sure the terminal that runs the command doesn't get closed during the whole testing process!**
8. Wait until all the data has been replayed and the system already process the data. To know if the test is done, see if the tcpreplay command is finished on the `tcpreplay` terminal and waits for about one minute.
9. Get the data for intrusion detection result on IDS dashboard's MongoDB with connection string `mongodb://mongoadmin:secret@[MONITORING_IP]:27017`. The collected data is on 'network-intrusion-detection' database with 'dos' collection for DoS and 'port_scan' collection for PortScan. You can use MongoDB Compass to connect to the database and download the collections' data as .csv files.
10. Run `terraform destroy -var-file="./variables/general.tfvars.json" -var-file="./variables/specs/system-name.tfvars.json"` to delete the system's infrastructure. Change system-name to the system you want to test. **Make sure to destroy then apply again before doing another testing**.