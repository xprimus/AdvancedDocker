FROM    ubuntu:14.04

MAINTAINER Xavier Primus <xjprimus@ncsu.edu>

#Install socat
RUN 	apt-get update
RUN 	apt-get install -y socat
RUN 	echo "This is the content of the file." > output.txt

#Run socat on exposed port 9001 and pipe the contents of the output file
CMD 	socat TCP4-LISTEN:9001,fork SYSTEM:'cat output.txt'
