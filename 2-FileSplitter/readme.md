# Message Flow Description
- File Input Node Configured to take CSV file but could be configured to take any file with specified delimiter character pre-deployment
- File Input Node is configured to be delimited with new lines
- File Input Sends records to compute node for its file name to be specified, then straight to the File Output Node to be written on disk
- File Output Node is also configured with same delimiter with option Records Are Delimited

# How To Run
- create /in and /out directories in project main directory
- Deploy on server of your choice 
- Place a file names in.csv in the /in directory
- Notice /out as the majic happens