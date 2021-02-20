# Remailer Assay Script  
Mixmaster Remailer 24h Chronograph/Continuity Analysis  
    
<p>
<img src="/images/analysis.png" width="1218" height="1615">  
</p>
```
#!/bin/bash
#
# Remailer-Proc-Pinger-Assay.sh  v1.11
#
# Script to create the Remailer-Proc-Pinger-Assay.sh stat records page
# A message sent hourly to every remailer (chain: <remailer to test>,<your remailer>)
# by this script at ??:01 hours to determine actual turn around
# time and continuity.  A 'user assay00' needs to be created.  The
# user assay00 does not use procmail and should not be setup for procmail.  The mail is
# found in /var/mail/assay00.
#
# The 'Remailer-Proc-Pinger-Assay.sh' script should be run in a folder: '/etc/assay00/'
#
# To 'exclude a remailer from the assay', place its short name in the file: 'Remailer-Proc-Pinger.blk'
#
# This script is executed every minute to keep the webpage current with this cronjob:
#'*/1 * * * * /etc/assay00/Remailer-Proc-Pinger-Assay.sh &> /dev/null'
#
#'-----------------------------------------------------------------------------------------'#
#

#Program structure information
#
#Remailer-Proc-Pinger-Assay-Mailer routine
#
#message subject:
# subject: banana Tue 01 Nov 2016 14:00 19
#            |     |   |   |   |    |    |
#            |     |   |   |   |    |    Cur GMT HR
#            |     Cur Cur Cur Cur  Cur Loc Tm
#            rem   DA  DA  MO  YR
#
# Script to mail hourly the records used by Remailer-Proc-Pinger-Assay.sh
# to create the assay.html webpage.  A user assay00 needs to be created.
# The user assay00 does not use Procmail.  The mail is found in
# /var/mail/assay00.
#
# A message is sent hourly to every remailer (chain: <remailer>,inwtx)
# to determine actual turn around time and continuity.
#
# The way to set up a middle remailer to act as an exit and send messages to a particular email address:
# Remailer files:
# dest.alw
# List of addresses to which Mixmaster will deliver, even in middleman mode (DESTALLOW).
# dest.alw.nonpublished
# Similar to dest.alw, with the only difference that this list is not published in remailer-conf replies (DESTALLOW2).
#
# 'dest.alw.nonpublished entries:'
# 'assay00@<yourRemailerDN>'
#
# These three parameters below must be modified:
#   MyShortNm="<your remailer short name>"  example: inwtx or austria
#   MyDN="<your remailer domain name>"      example: dizum.com or mixharbor.xyz
#   webpgpath="/path/to/html" folder        example: (nginx) /var/www/html
#
# Example of generated output heaaders:
#Chain: redjohn,inwtx
#To: assay00@inwtx.net
#Subject: Test a
#
#'--------------------------------------------------------------------------------------'#

export PATH=$PATH:/etc
export PATH=$PATH:/var/www
export PATH=$PATH:/var/www/html
export PATH=$PATH:/usr/bin

filePath=${0%/*} # current file path $filePath/

MyShortNm="your remailer short name"
MyDN="your remailer domain name"

webpgpath="/var/www/html"

assayid="assay00"
procit="y"
webpgtmpnm="temp.html"
webpgtmpnm2="temp2.html"
webpgnm="$assayid.html"
webpgnm2="fail.html"
mkred="<font color=\$FF0000><b>"  # must put one space after $mkred or outside of " "
nored="</b></font>"

if [ ! -e $filePath/Remailer-Proc-Pinger.blk ]; then
   echo "slow" > $filePath/Remailer-Proc-Pinger.blk
   printf '\x89\x50\x4E\x47\x0D\x0A\x1A\x0A\x00\x00\x00\x0D\x49\x48\x44\x52\x00\x00\x00\x12\x00\x00\x00\x0B\x08\x06\x00\x00\x00\x72\x17\xDD\x04\x00\x00\x00\x01\x73\x52\x47\x42\x00\xAE\xCE\x1C\xE9\x00\x00\x00\x04\x67\x41\x4D\x41\x00\x00\xB1\x8F\x0B\xFC\x61\x05\x00\x00\x00\x09\x70\x48\x59\x73\x00\x00\x0E\xC2\x00\x00\x0E\xC2\x01\x15\x28\x4A\x80\x00\x00\x00\xD8\x49\x44\x41\x54\x38\x4F\x63\xFC\xBA\xF2\xD3\x7F\x06\x2A\x00\x26\x28\x4D\x31\xA0\x93\x41\x17\x96\x32\x70\x87\x97\x40\xF0\xEC\x2B\x50\xC1\x97\x0C\x53\x6B\x4B\x18\x72\x2F\x40\xB9\x50\x80\x6A\xD0\x8B\xBD\x0C\xCE\xC8\x9A\x0C\xA2\x19\xBE\xAE\xEC\x01\xE2\x04\x86\xA4\x47\x2F\x19\xEE\x02\x85\x76\xCD\x5E\xCA\xC0\x60\x69\x08\x91\x47\x02\x40\x83\x40\x9A\xAE\x30\xE4\x82\x0C\x98\xCA\xC0\x30\x0B\xA4\x31\x55\x07\x22\x0B\x05\x77\xB7\xED\x62\x60\x08\x76\x66\x60\xD8\xD6\xC7\xB0\xD9\xB4\x88\x21\x5B\x0A\x2A\x81\x04\x90\x62\x0D\x64\xD8\x02\x86\x79\x6A\x9E\x0C\x97\x9A\x9D\x19\x94\x21\x82\x40\x43\xFA\x18\xD2\x18\xA2\x19\xF6\x7A\x89\x03\x5D\x53\xC2\x10\xB8\x07\x2A\xC1\x20\xC5\xD0\x35\x11\x68\xA8\x04\x84\x87\x19\xFD\x20\xEF\x81\x5C\x06\x34\x0C\xE4\x02\xBD\x85\xCF\xA0\x12\x86\x0C\xEB\x57\x46\x33\xB8\x81\x98\xC0\xB0\xCB\x05\x1A\x3E\xD9\x00\x2C\x01\x06\xC3\x36\x1D\x31\x30\x00\x00\xF0\x90\x4B\xBF\xD1\xA0\x39\x5C\x00\x00\x00\x00\x49\x45\x4E\x44\xAE\x42\x60' > $webpgpath/purple.png  # purple.png
   printf '\x89\x50\x4e\x47\x0d\x0a\x1a\x0a\x00\x00\x00\x0d\x49\x48\x44\x52\x00\x00\x00\x10\x00\x00\x00\x0b\x08\x06\x00\x00\x00\x76\xe2\x0d\x39\x00\x00\x00\x01\x73\x52\x47\x42\x00\xae\xce\x1c\xe9\x00\x00\x00\x04\x67\x41\x4d\x41\x00\x00\xb1\x8f\x0b\xfc\x61\x05\x00\x00\x00\x09\x70\x48\x59\x73\x00\x00\x0e\xc4\x00\x00\x0e\xc4\x01\x95\x2b\x0e\x1b\x00\x00\x00\x51\x49\x44\x41\x54\x28\x53\x63\xfc\x7f\xe2\xc0\x7f\x06\x0a\x00\x13\x94\x26\x1b\xd0\xc6\x80\x77\x21\x29\x60\xfc\x17\x8d\x8f\x0d\xe0\x0c\x03\x74\x0d\x42\x6b\xe6\x40\x59\xa8\x00\xa7\x17\x90\x35\xe0\xd2\x0c\x02\x38\x0d\x40\x76\x01\x2e\xe7\x83\x00\xce\x30\x00\x01\x90\xcd\x30\xdb\x49\x0e\x03\x62\x01\x6d\xa2\x91\x78\xc0\xc0\x00\x00\x21\x4f\x21\x3b\xf9\x08\x7d\x7f\x00\x00\x00\x00\x49\x45\x4e\x44\xae\x42\x60' > $webpgpath/redx.png  # redx.png
   printf '\x89\x50\x4E\x47\x0D\x0A\x1A\x0A\x00\x00\x00\x0D\x49\x48\x44\x52\x00\x00\x00\x10\x00\x00\x00\x0B\x08\x06\x00\x00\x00\x76\xE2\x0D\x39\x00\x00\x00\x01\x73\x52\x47\x42\x00\xAE\xCE\x1C\xE9\x00\x00\x00\x04\x67\x41\x4D\x41\x00\x00\xB1\x8F\x0B\xFC\x61\x05\x00\x00\x00\x09\x70\x48\x59\x73\x00\x00\x0E\xC4\x00\x00\x0E\xC4\x01\x95\x2B\x0E\x1B\x00\x00\x00\x51\x49\x44\x41\x54\x28\x53\x63\xFC\x0F\x04\x0C\x14\x00\x26\x28\x4D\x36\xA0\x8D\x01\xEF\x64\x55\xC1\xF8\xEF\xC9\x83\x28\x7C\x6C\x00\x67\x18\xA0\x6B\x10\x7A\x7C\x1B\xCA\x42\x05\x38\xBD\x80\xAC\x01\x97\x66\x10\xC0\x69\x00\xB2\x0B\x70\x39\x1F\x04\x70\x86\x01\x08\x80\x6C\x86\xD9\x4E\x72\x18\x10\x0B\x68\x13\x8D\xC4\x03\x06\x06\x00\x36\x45\x22\x4F\xFE\xAA\xD6\x77\x00\x00\x00\x00\x49\x45\x4E\x44\xAE\x42\x60' > $webpgpath/whitex.png  # whitex.png
fi

## BEGIN This was the former Remailer-Proc-Pinger-Assay-Mailer.sh routine that was incorporated here.

if [[ $(date +"%M") = "01" ]]; then
   statDN="http://sec3.net/echolot/mlist2.txt"
   ##statDN="http://www.mixmin.net/echolot/mlist2.txt"

   #'======================'
   #' remailer stats begin '
   #'======================'

   wget --no-check-certificate --timeout=15 -t 1 $statDN -O $filePath/IPPAMmlist.txt  # get mlist.txt
   grep "\$remailer" $filePath/IPPAMmlist.txt > $filePath/IPPAMlist2.txt              # extract "$remailer" lines

   while read line1; do
         RemailerAddress=$(cut -d'<' -f2 <<<$line1 | cut -d'>' -f1)
         ShortName=$(cut -d'"' -f2 <<<$line1)

         if [[ $(cat $filePath/starex.txt) =~ $ShortName ]]; then
            continue
         fi

         echo "Chain: $ShortName,$MyShortNm" > $filePath/Remailer-Proc-Pinger-Assay-Mailer.txt

         echo "To: $assayid@$MyDN" >> $filePath/Remailer-Proc-Pinger-Assay-Mailer.txt
         echo "Subject: $ShortName $(date +"%a %d %b %Y %H:%M")" "$(date -u +"%H")" >> $filePath/Remailer-Proc-Pinger-Assay-Mailer.txt  # Mon 16 Sep 12:07 2019 17
         echo "" >> $filePath/Remailer-Proc-Pinger-Assay-Mailer.txt
         echo "$ShortName $(date +"%a %d %b %H:%M %Y")" "$(date -u +"%H")" >> $filePath/Remailer-Proc-Pinger-Assay-Mailer.txt

         /usr/bin/mixmaster $filePath/Remailer-Proc-Pinger-Assay-Mailer.txt --send --mail
         sleep 3
   done< $filePath/IPPAMlist2.txt

   exit 0
fi
## END This was the former Remailer-Proc-Pinger-Assay-Mailer.sh routine that was incorporated here.


## BEGIN This was the former the hour24.sh script routine that was incorporated here.
if [[ $(date +"%M") = "57" ]]; then
   remcnt=$(expr $(wc -l $filePath/statsav.txt | awk '{print $1}') / 24)  # get cnt of remailers

   s24cnt=24

   while [[ $i -lt $remcnt ]]; do
         i=$[$i+1]
         sed -n $s24cnt'p' < $filePath/statsav.txt >> $filePath/statsav.txt2  # sav last rec of each remailer 24hr groups
         s24cnt=$(( s24cnt + 24 ))  # bump to next remailer 24hr group
   done

   rem24cnt=$(( remcnt * 24 ))

   if [ -e $filePath/statsav.txt2 ] && [[ $(cat $filePath/statsav.txt2 | wc -l) -gt rem24cnt ]]; then  # purge top 24 recs if over remcnt * 24 (hrs) (-gt rem24cnt not $rem2$
      sed -i 1,$remcnt'd' $filePath/statsav.txt2  # purge top 24 recs
   fi

   cat /dev/null > $filePath/statsav.txt3  # empty statsav.txt3

   while read line1; do
         if [[ $(awk '{ print $8 }' <<<$line1) == "x" ]]; then
            echo $(awk '{ print $7" "$8 }' <<<$line1) >> $filePath/statsav.txt3
         fi
   done< $filePath/statsav.txt2

   sort -o $filePath/statsav.txt3 $filePath/statsav.txt3

   cat $filePath/statsav.txt3 | sort | uniq -c | sort -nk1 > $filePath/statsav.x  # save next hours display of previous 24 hr missed messages for next hr's display

   exit 0
fi
## END  This was the former the hour24.sh script routine that was incorporated here.



if [[ $(date +"%M") = "03" ]] || \
   [[ $(date +"%M") = "05" ]] || \
   [[ $(date +"%M") = "10" ]] || \
   [[ $(date +"%M") = "15" ]] || \
   [[ $(date +"%M") = "20" ]] || \
   [[ $(date +"%M") = "25" ]] || \
   [[ $(date +"%M") = "30" ]] || \
   [[ $(date +"%M") = "35" ]] || \
   [[ $(date +"%M") = "40" ]] || \
   [[ $(date +"%M") = "45" ]] || \
   [[ $(date +"%M") = "50" ]] || \
   [[ $(date +"%M") = "55" ]]; then
   :
else
   exit 0
fi

function DeleteFiles {
      rm $filePath/stat1.txt 2> /dev/null
      rm $filePath/stat2.txt 2> /dev/null
      rm $filePath/stat3.txt 2> /dev/null
      rm $filePath/stat4.txt 2> /dev/null
      rm $filePath/stat5.txt 2> /dev/null
      rm $filePath/stat6.txt 2> /dev/null
      rm $filePath/stat7.txt 2> /dev/null
      rm $filePath/stat8.txt 2> /dev/null
      rm $filePath/stat9.txt 2> /dev/null
      rm $filePath/stat10.txt 2> /dev/null
      rm $filePath/stat11.txt 2> /dev/null
      rm $filePath/stat12.txt 2> /dev/null
} ## DeleteFiles

DeleteFiles


#'================================================================'#
echo "=== Create GMT string beginning with this hour's GMT time ==="
#'================================================================'#
gmtnow=$(date -u +'%H')  # get current gmt hour

unset arrayIPP

j=24
i=$gmtnow

while [ $i -gt -1 ]; do
      ((j--))
      if [[ j -lt 0 ]]; then  # loop only 24 times
         break
      fi

      arrayIPP+=("$(printf "%02d" $(echo $i | sed 's/^0*//'))")

      i=$[$(echo $i | sed 's/^0*//')-1]

      if [[ $i -eq -1 ]]; then
         i=23
      fi
done

echo ${arrayIPP[*]}  # display all of the items in the array


#'========================================================='#
echo "=== Retrieve base records from assay mail records ==="
#'========================================================='#

if [ -e /var/mail/$assayid ] && [[ $(cat /var/mail/$assayid | wc -l) -gt 0 ]]; then
   cat /var/mail/$assayid > $filePath/Remailer-Proc-Pinger-Assay.txt  # transfer mail from echolot mail file
   cat /dev/null > /var/mail/$assayid                              # purge old mail if not testing
else
   if [ ${procit:+1} ]; then   # something in procit? then force pprocessing without data
      cat /var/mail/$assayid > $filePath/Remailer-Proc-Pinger-Assay.txt  # transfer mail from echolot mail file
      cat /dev/null > /var/mail/$assayid                              # purge old mail if not testing
   else
   exit 0
   fi
fi


#'==========================================='#
echo "=== Combine subject: and Date: lines ==="
#'==========================================='#

grep -ie "subject:" $filePath/Remailer-Proc-Pinger-Assay.txt > $filePath/stat1.txt # pull out subject: lines
grep -ie "Date:" $filePath/Remailer-Proc-Pinger-Assay.txt > $filePath/stat2.txt    # pull out Date: lines
pr -m -t -J $filePath/stat1.txt $filePath/stat2.txt > $filePath/stat3.txt # concat subject: and Date: lines from stat1.txt and stat2.txt

sed -i 's/\t/  /g' $filePath/stat3.txt  # replace \t with 2 spaces
sed -i 's/"//g' $filePath/stat3.txt     # remove all double quotes
sed -i '/^$/d' $filePath/stat3.txt      # delete blank lines
sed -r '/^.{,65}$/d' $filePath/stat3.txt >> $filePath/Remailer-Proc-Raw-Recs.txt  # retain lines > 65 chars

sort -k2 -k4 -k8 $filePath/Remailer-Proc-Raw-Recs.txt > $filePath/stat4.txt  # sort by shortname, date, and gmt time

awk '!a[$0]++' $filePath/stat4.txt > $filePath/Remailer-Proc-Raw-Recs.txt    # remove duplicate records

sed -i 's/ 1 / 01 /g' $filePath/Remailer-Proc-Raw-Recs.txt                   # replace single num day with leading zero
sed -i 's/ 2 / 02 /g' $filePath/Remailer-Proc-Raw-Recs.txt                   # replace single num day with leading zero
sed -i 's/ 3 / 03 /g' $filePath/Remailer-Proc-Raw-Recs.txt                   # replace single num day with leading zero
sed -i 's/ 4 / 04 /g' $filePath/Remailer-Proc-Raw-Recs.txt                   # replace single num day with leading zero
sed -i 's/ 5 / 05 /g' $filePath/Remailer-Proc-Raw-Recs.txt                   # replace single num day with leading zero
sed -i 's/ 6 / 06 /g' $filePath/Remailer-Proc-Raw-Recs.txt                   # replace single num day with leading zero
sed -i 's/ 7 / 07 /g' $filePath/Remailer-Proc-Raw-Recs.txt                   # replace single num day with leading zero
sed -i 's/ 8 / 08 /g' $filePath/Remailer-Proc-Raw-Recs.txt                   # replace single num day with leading zero
sed -i 's/ 9 / 09 /g' $filePath/Remailer-Proc-Raw-Recs.txt                   # replace single num day with leading zero
sed -i 's/,  /, /g' $filePath/Remailer-Proc-Raw-Recs.txt                     # replace single num day with leading zero

sed -i 's/ (CST)//g' $filePath/Remailer-Proc-Raw-Recs.txt


#'========================================'#
echo "=== Remove recs older than 24 hrs ==="
#'========================================'#

cat /dev/null > $filePath/stat4.txt

#message subject:
#subject: banana Tue 01 Nov 2016 14:00 19
#           |     |   |   |   |    |    |
#           |     |   |   |   |    |    Cur GMT HR
#           |     Cur Cur Cur Cur  Cur Loc Tm
#           rem   DA  DA  MO  YR

#   1       2     3  4   5   6     7   8     9    10  11 12   13     14      15    16
#subject: banana Tue 01 Nov 2016 14:00 19  Date: Tue, 01 Nov 2016 14:34:35 -0500 (CDT)

sleep 5  # wait 5 sec to get past top of hr

while read line1; do
      varIPPt=$(date +%s)  # get cur tm or may jump 1 sec and bypass good recs
      varIPP1=$(awk '{print $4" "$5" "$6" "$7}' <<<$line1)  # extract '01 Nov 2016 14:00'
      varIPP2=$(date -d "$varIPP1" +%s)     # record epoch date
      varIPP=$(( $varIPPt - $varIPP2 ))  # seconds difference btwn record and current dates

      if [[ $varIPP -ge 86400 ]]; then  # 86400 = 24 hours - 60*60*24
         continue  # drop dates >= 24 hrs
         else
         echo "$line1" >> $filePath/stat4.txt # retain last 24 hr lines only
      fi
done< $filePath/Remailer-Proc-Raw-Recs.txt

cat $filePath/stat4.txt > $filePath/Remailer-Proc-Raw-Recs.txt  # put recs within last 24 hrs back into Remailer-Proc-Rot-Recs.txt for bkup


#'======================================'#
echo "=== Calculate turn around times ==="
#'======================================'#

#new record
#   1       2     3  4   5   6     7   8      9       10   11   12 13   14     15      16
#subject: banana Tue 01 Nov 2016 14:00 19 epochdate  Date: Tue, 01 Nov 2016 14:34:35 -0500

reccnt=0

while read line1; do
      sname=$(awk '{print $2}' <<<$line1)       # remailer name
      sgmt=$(awk '{print $8}' <<<$line1)        # gmt send hour
      senttime1=$(awk '{print $4" "$5" "$6" "$7}' <<<$line1)                        # = 21 Oct 2016 00:00
      recvdtime1=$(awk '{print $11" "$12" "$13" "$14}' <<<$line1 | cut -d':' -f1-2) # = 21 Oct 2016 02:04
      savdtIPP=$(awk '{print $3" "$4" "$5" "$6}' <<<$line1)                         # save sent date in subject = Fri 21 Oct 2016
      savtmIPP=$(awk '{print $7}' <<<$line1)                                        # save sent time in subject = 00:00
      SENTTIME=$(date +%s -d "$senttime1")      # cvt to epoch time
      RECVDTIME=$(date +%s -d "$recvdtime1")    # cvt to epoch time
      MINUTES=$(( ( $RECVDTIME - $SENTTIME ) )) # sub epoch times

      printf '%-4s %-3s %-4s %-3s %-5s %-6s %-11s %-5s\n' $(printf "$savdtIPP $savtmIPP $sgmt $sname %d:%02d\n" $(($MINUTES/3600)) $(($MINUTES%3600/60))) >> $filePath/stat5.txt
      echo "$savdtIPP $savtmIPP $sgmt $sname" >> $filePath/stat11.txt
done< $filePath/Remailer-Proc-Raw-Recs.txt         # read file line by line


#'============================================================='#
echo "=== Extract each remailer's records to correctly sort ==="
#'============================================================='#

cat /dev/null > $filePath/stat9.txt
cat /dev/null > $filePath/stat10.txt

varUReIPP=$(sort -u -k7,7 $filePath/stat5.txt | awk '{ print $7 }')  # get single remailer names list
echo "$varUReIPP" > $filePath/stat6.txt  # put single remailer names list into file

while read "line1"; do
      if [[ $(cat $filePath/Remailer-Proc-Pinger.blk) =~ "$line1" ]]; then  # skip /b/ lines and names in Remailer-Proc-Pinger.blk file
         continue
      fi

      grep -w "$line1" $filePath/stat5.txt > $filePath/stat7.txt              # extract all of this <$varUReIPP> remailer records
      sort -rk 6,6 $filePath/stat7.txt | sort -rk 2,2 > $filePath/stat8.txt   # sort the remailer by gmt hr and day
      sed -i '25,$ d' $filePath/stat8.txt                                     # trunc file to 24 recs
      savshortnm=$(sed -n 1p $filePath/stat8.txt | awk '{print $7}')          # save remailer short name

      cat /dev/null > $filePath/stat9.txt   # clear file

      for i in "${arrayIPP[@]}"; do  # find and write in dummy recs for missing records
          rectest=""
          rectest=$(grep -e " $i     " $filePath/stat8.txt)  # pull from sorted 'Fri  21  Oct  2016 01:00 06     anon        0:49' recs per array gmt time pos 6

          if [[ ${#rectest} -gt 0 ]]; then
             echo "$rectest" >> $filePath/stat9.txt
             else
             echo "ddd  dd  mmm  yyyy hh:mm $i     $savshortnm            x" >> $filePath/stat9.txt

#      1         2       3  4   5   6     7   8   9        10               11
#message-lost epochdate Sat 12 Nov 2016 13:00 19  19     austria            x
             vardtIPP=$(date -u -d "$(date +"%Y-%m-%d $i")"  +%s)  # create cur date/cur gmt from array and put in x rec
             echo "message-lost $vardtIPP $(date '+%a %d %b %Y %H:00' | cut -d':' -f1-2) $(date -u +'%H')  $i  $savshortnm  x" >> $filePath/Remailer-Proc-AWOL.txt # sav for logging section
          fi
      done

      sed -i '25,$ d' $filePath/stat9.txt              # trunc file to 24 recs

      cat $filePath/stat9.txt >> $filePath/stat10.txt  # accumulate all remailer results
done< $filePath/stat6.txt

sed -i '/^$/d' $filePath/stat10.txt  # delete empty lines

cat $filePath/stat10.txt > $filePath/statsav.txt

#      1         2       3  4   5   6     7   8   9        10               11
#message-lost epochdate Sat 12 Nov 2016 13:00 19  19     austria            x

# 7 day fail return log
if [[ $(date +"%M") = "00" ]]; then  # only write out one rec at top of hour
#     1           2      3  4   5   6     7   8   9     10     11
#message-lost epochdate Mon 21 Nov 2016 15:00 21  21  lulunga  x
   while read "line1"; do
         if [[ $(cat $filePath/Remailer-Proc-AWOL.txt | wc -l) -gt 0 ]] && [[ $(awk '{ print $9 }' <<<$line1) -eq ${arrayIPP[7]} ]] && [[ $(awk '{ print $11 }' <<<$line1) -eq "x" ]]; then
            echo "$(awk '{ print $10" lost - sent: "$5" "$4" "$9"00 Z" }' <<<$line1)  $(date +%s)" >> $filePath/Remailer-Proc-AWOL.log  # create error rec
         fi
   done< $filePath/Remailer-Proc-AWOL.txt
fi


#Last 7 Days History
sort -o $filePath/Remailer-Proc-AWOL.log $filePath/Remailer-Proc-AWOL.log  # sort in place
sed -i '$!N; /^\(.*\)\n\1$/!P; D' $filePath/Remailer-Proc-AWOL.log      # remove blank lines

#     1           2      3  4   5   6     7   8   9     10     11
#message-lost epochdate Mon 21 Nov 2016 15:00 21  21  lulunga  x
#go thru stat11.txt and del any recs in Remailer-Proc-AWOL.log with same remailername/epoch
#echo "=== Delete found records listed in No Response History ==="
# 1  2   3   4     5   6   7  stat11.txt
#Mon 21 Nov 2016 00:00 06 anon

while read "line1"; do  # create epoch date for lines in stat11.txt
      vargt=$(awk '{print $6}' <<<$line1)
      vardtIPP2=$(date -u -d "$(date +"%Y-%m-%d $vargt")"  +%s)
      echo "$line1 $vardtIPP2" >> $filePath/stat12.txt
done< $filePath/stat11.txt

# This routine needs perfecting
#   1      2       3    4  5   6   7     8     Remailer-Proc-AWOL.log
#hermetix lost - sent: Nov 18 0000 Z  GMTepoch

##while read "line1"; do  # search thru Remailer-Proc-AWOL.log to del later found error recs
##      word1TPP=$(awk '{print $1}') # get remailer name
##      word2TPP=$(awk '{print $8}') # get GMTepoch
##      if [[ $(grep $word1TPP $filePath/stat12.txt | grep -c $word2TPP) -gt 0 ]]; then
##         line3TPP=$(grep $word1TPP $filePath/Remailer-Proc-AWOL.log | grep $word2TPP $filePath/Remailer-Proc-AWOL.log)
##         echo "$line3TPP $(date)" >> dummy.txt
##         sed -i "/$line3TPP/d" $filePath/Remailer-Proc-AWOL.log
##      fi
##done< $filePath/Remailer-Proc-AWOL.log
##

cat /dev/null > $filePath/Remailer-Proc-AWOL.txt  # AWOL records have been processed and no longer needed

echo "<table border=\"1\" cellpadding=\"2\" cellspacing=\"0\">" > $filePath/$webpgtmpnm2
echo "<tr><td align=\"center\" bgcolor=\"E7DFAD\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;No Response History - Last 7 Days&thinsp;</font></td></tr>" >> $filePath/$webpgtmpnm2
echo "<tr><td align=\"left\" bgcolor=\"FFC8C0\"><font face=\"Lucida Console\" size=\"2\" color=\"000040\">" >> $filePath/$webpgtmpnm2
hitIPP=0

while read "line1"; do               # write out 'No Response History - Last 7 Days' lines
      if [[ $(date -u +%s) -ge $(( $( awk '{print $9}' <<<$line1 ) + 604800 )) ]]; then  # remove error msgs > 7 days and create webpge list - mins in 7 da = 604800.00
         sed -i "s/$line1//g" $filePath/Remailer-Proc-AWOL.log                              # delete expired error line > 7 days
         sed -i '/^$/d'  $filePath/Remailer-Proc-AWOL.log                                   # remove blank line left behind
         continue
      fi

      varIPPlog=$(printf '%-14s %-s %-s %-s %-s %-s %-s %-s\n' $(awk '{ print $1" "$2" "$3" "$4" "$5" "$6" "$7" "$8 }' <<<$line1) | sed 's/ /\&nbsp;/g')
      echo "&thinsp;$varIPPlog&thinsp;<br>" >> $filePath/$webpgtmpnm2
      hitIPP=1
done< $filePath/Remailer-Proc-AWOL.log

if [[ $hitIPP -eq 0 ]]; then
   echo "&thinsp;none<br>" >> $filePath/$webpgtmpnm2
   else
   sed -i "s/none//g" $filePath/Remailer-Proc-AWOL.log    # delete expired error line > 7 days
   sed -i '/^$/d'  $filePath/Remailer-Proc-AWOL.log       # remove blank line left behind
fi

sort -o $filePath/Remailer-Proc-AWOL.log $filePath/Remailer-Proc-AWOL.log && sed -i '$!N; /^\(.*\)\n\1$/!P; D' $filePath/Remailer-Proc-AWOL.log  # del dup lines

echo "</font></td></tr>" >> $filePath/$webpgtmpnm2
echo "</tr>" >> $filePath/$webpgtmpnm2
echo "</table>" >> $filePath/$webpgtmpnm2


#'==========================================='#
echo "=== Create GMT HTML time header only ==="
#'==========================================='#

# create title line
echo "<table border=\"0\" cellpadding=\"2\" cellspacing=\"4\">" > $filePath/$webpgtmpnm

echo "<tr>" >> $filePath/$webpgtmpnm
echo "<!-- top header -->" >> $filePath/$webpgtmpnm
#echo "<td align=\"left\" bgcolor=\"EBEAD9\"><font face=\"Verdana\" size=\"2\" color=\"000040\">Mixmaster Remailer 24<small>h</small> Chronograph/Continuity Analysis&nbsp;&thinsp;&nbsp;-&nbsp;$(date -u +"%F - %H:%M %Z")&nbsp;-&nbsp;$(date +"%H:%M %Z")</font></td>" >> $filePath/$webpgtmpnm
echo "<td align=\"left\" bgcolor=\"EBEAD9\"><font face=\"Verdana\" size=\"2\" color=\"000040\">Mixmaster Remailer 24<small>h</small> Chronograph/Continuity Analysis&nbsp;&nbsp;$(date -u +"%F - %H:%M %Z")&nbsp;-&nbsp;$(date +"%H:%M %Z")</font></td>" >> $filePath/$webpgtmpnm

echo "</tr>" >> $filePath/$webpgtmpnm
echo "</table>" >> $filePath/$webpgtmpnm

# create gmt num string line
echo "<table border=\"1\" cellpadding=\"2\" cellspacing=\"0\">" >> $filePath/$webpgtmpnm
echo "<tr>" >> $filePath/$webpgtmpnm
echo "<!-- GMT header -->" >> $filePath/$webpgtmpnm
echo "<td align=\"center\" bgcolor=\"E7DFAD\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;Post Time GMT:&thinsp;</font></td>" >> $filePath/$webpgtmpnm

clrcurgmt="FFCF00"

aaIPP=0

for i in `seq 0 23`; do
    ((aaIPP++))

    if [[ $aaIPP -eq 7 ]]; then  # chg col 7 header to rose
       clrcurgmt="FFC8C0"
    fi

    gmtnow2=${arrayIPP[$i]}
    gmtline=$(sed 's/gmtinc/'$gmtnow2'/g' <<<"<td align=\"center\" bgcolor=\"$clrcurgmt\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&nbsp;&nbsp;gmtinc&nbsp;&nbsp;</font></td>")
    echo "$gmtline" >> $filePath/$webpgtmpnm
    clrcurgmt="f0f0f0"
done

echo "<td align=\"center\" bgcolor=\"E7DFAD\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;Hi&thinsp;</font></td>" >> $filePath/$webpgtmpnm # high title
echo "<td align=\"center\" bgcolor=\"E7DFAD\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;Lo&thinsp;</font></td>" >> $filePath/$webpgtmpnm # low  title
echo "<td align=\"center\" bgcolor=\"F5A9F2\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&nbsp;&lt 24&nbsp;</font></td>" >> $filePath/$webpgtmpnm # previous 24  title
clrcurgmt="f0f0f0"

echo "</tr><tr>" >> $filePath/$webpgtmpnm


#'========================================================='#
echo "=== Create remailer HTML records from base records ==="
#'========================================================='#

function ChgRowColor {
         if [[ $bgcolor1 = "EBEAD9" ]]; then   # flip flop bkgnd color
            bgcolor1="f0f0f0"
            savbgcolor1=$bgcolor1
            else
            bgcolor1="EBEAD9"
            savbgcolor1=$bgcolor1
         fi
         }

bgcolor1="f0f0f0"
savbgcolor1="f0f0f0"
sname=""

sed -i '/^$/d' $filePath/stat5.txt  # remove any blank lines
aaIPP=0
hiIPP=0
loIPP=0
highprevnameortime="00:00"
lowprevnameortime="99:99"

function HiLoAvg(){      # pause function
         hiIPP=$([ $(tr -d \: <<<$highprevnameortime) -gt $(tr -d \: <<<$nameortime) ] && echo $(tr -d \: <<<$highprevnameortime) || echo $(tr -d \: <<<$nameortime)) # hi test

         if [ $(tr -d \: <<<$nameortime) -gt $(tr -d \: <<<$highprevnameortime) ]; then
            highprevnameortime=$nameortime
         fi

         loIPP=$([ $(tr -d \: <<<$lowprevnameortime) -lt $(tr -d \: <<<$nameortime) ] && echo $(tr -d \: <<<$lowprevnameortime) || echo $(tr -d \: <<<$nameortime)) # lo test

         if [ $(tr -d \: <<<$nameortime) -lt $(tr -d \: <<<$lowprevnameortime) ]; then
            lowprevnameortime=$nameortime
         fi
} # HiLoAvg

while read line1; do  # build time string line for each remailer BEGIN
      ((aaIPP++))

      if [[ ${#sname} -eq 0 ]]; then  # 1st time thru (?)
         echo "<!-- remailer header -->" >> $filePath/$webpgtmpnm
         ChgRowColor
         nameortime=$(awk '{print $7}' <<< $line1)  # get remailer name
         varrname=$(grep -w $nameortime $filePath/IPPAMlist2.txt)  # get remailer name stats rec if present

         remcolor="000040"
         if [[ ! $varrname == "" ]]; then  # change a remailer exit name to red
            if [[ $varrname =~ "middle" ]]; then
               remcolor="000040"
            else
               remcolor="FF0080"
            fi
         fi

         echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"$remcolor\">&thinsp;$nameortime&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build time

         nameortime=$(awk '{print $8}' <<< "$line1")  # get remailer 1st turn time

         if [[ $nameortime == "0:00" ]]; then  # make 1 min if < a min
            nameortime="0:01"
         fi

         if [[ $nameortime == "x" ]]; then
            echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;$mkred$nameortime$nored&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build time
         else
            echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;$nameortime&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build time
            highprevnameortime="00:00"
            lowprevnameortime="99:99"
            HiLoAvg
         fi

         sname=$(awk '{print $7}' <<<$line1)
         continue
      fi

      if [[ $sname != $(awk '{print $7}' <<<$line1) ]]; then  # see if remailer name change - not 1st time through - see above
      # remailer name has changed
         hiIPPout=$(sed ':a;s/\B[0-9]\{2\}\>/:&/;ta'<<<$hiIPP)
         loIPPout=$(sed ':a;s/\B[0-9]\{2\}\>/:&/;ta'<<<$loIPP)
         echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"800000\">&thinsp;$hiIPPout&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build high
         echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"800000\">&thinsp;$loIPPout&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build low

if true; then                                  
## build previous 24 hour records BEGIN
         pastmissed=0
         if [[ $(grep -wc $sname $filePath/statsav.x) -gt 0 ]]; then
            pastmissed=$(grep -w $sname $filePath/statsav.x | awk '{ print $1 }')
         fi

         if [[ $pastmissed == "0" ]]; then
            echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;$pastmissed&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build previous 24 missed count
         else
            echo "<td align=\"center\" bgcolor=\"F5A9F2\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;$pastmissed&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build previous 24 missed count
         fi
## build previous 24 hour records END
fi                                            

         hiIPP=0
         loIPP=0
         highprevnameortime="00:00"
         lowprevnameortime="99:99"
         ChgRowColor
         echo "</tr><tr>" >> $filePath/$webpgtmpnm     # divide html lines
         echo "<!-- remailer header -->" >> $filePath/$webpgtmpnm
         nameortime=$(awk '{print $7}' <<<$line1)  # get remailer name

         varrname=$(grep -w $nameortime $filePath/IPPAMlist2.txt)  # get remailer name stats rec if present
         remcolor="000040"
         if [[ ! $varrname == "" ]]; then  # change a remailer exit name to red
            if [[ $varrname =~ "middle" ]]; then
               remcolor="000040"
            else
               remcolor="FF0080"
            fi
         fi

         echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"$remcolor\">&thinsp;$nameortime&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build time

         nameortime=$(awk '{print $8}' <<<"$line1")  # get remailer 1st turn line

         if [[ $nameortime == "0:00" ]]; then  # make 1 min if < a min
            nameortime="0:01"
         fi

         if [[ $nameortime == "x" ]]; then
            echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;$mkred$nameortime$nored&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build time
            else
            echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;$nameortime&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build time
            HiLoAvg
         fi

         aaIPP=1
         sname=$(awk '{print $7}' <<<$line1)
      else
      # remailer name has not changed
         nameortime=$(awk '{print $8}' <<<"$line1")  # get remailer turn time

         if [[ $nameortime == "0:00" ]]; then
            nameortime="0:01"
         fi

         if [[ $nameortime == "x" ]]; then
            if [[ $aaIPP -ge 7 ]]; then  # chg col 7 header to rose
               savbgcolor1=$bgcolor1
               bgcolor1="FFC8C0"
            fi

            echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;$mkred$nameortime$nored&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build time
            bgcolor1=$savbgcolor1
         else
            echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;$nameortime&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build time
            HiLoAvg
         fi
      fi
done< $filePath/stat10.txt  # build time string line for each remailer END


hiIPPout=$(sed ':a;s/\B[0-9]\{2\}\>/:&/;ta'<<<$hiIPP)  # build out last record hi/lo
loIPPout=$(sed ':a;s/\B[0-9]\{2\}\>/:&/;ta'<<<$loIPP)
echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"800000\">&thinsp;$hiIPPout&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build high
echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"800000\">&thinsp;$loIPPout&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build low

## build previous 24 hour records BEGIN
pastmissed=0
if [[ $(grep -wc $sname $filePath/statsav.x) -gt 0 ]]; then      # build out last column = past 24 hrs
   pastmissed=$(grep -w $sname $filePath/statsav.x | awk '{ print $1 }')
fi

if [[ $pastmissed == "0" ]]; then
   echo "<td align=\"center\" bgcolor=\"$bgcolor1\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;$pastmissed&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build previous 24 missed count
else
   echo "<td align=\"center\" bgcolor=\"F5A9F2\"><font face=\"Verdana\" size=\"2\" color=\"000040\">&thinsp;$pastmissed&thinsp;</font></td>" >> $filePath/$webpgtmpnm # build previous 24 missed count
fi
## build previous 24 hour records END

#                                              

#'================================='#
echo "=== Final HTML end records ==="
#'================================='#

echo "</tr><tr>" >> $filePath/$webpgtmpnm
echo "</tr></table>" >> $filePath/$webpgtmpnm

echo "<table align=left>" >> $filePath/$webpgtmpnm
echo "<tr>" >> $filePath/$webpgtmpnm
echo "<td><font face=\"Verdana\" size=\"1\" color=\"000040\">[Message sent :00 to every remailer (chain: &ltremailer&gt,$MyShortNm) to determine actual turn around time and continuity. (cycle time h:mm) (<font face=\"Verdana\" size=\"1\" color=\"FF0080\">name</font><font face=\"Verdana\" size=\"1\" color=\"000040\"> = exit) (<sub><img src=\"whitex.png\" HEIGHT="11" alt=\"x\"></sub> = pending) (<sub><img src=\"redx.png\" HEIGHT="11" alt=\"x\"></sub> = lost&gt6h) (<sub><img src=\"purple.png\" HEIGHT="11" alt=\"0\"></sub> = previous 24 hr loss)  (updt=5m)]</font></td>" >> $filePath/$webpgtmpnm
echo "</font>" >> $filePath/$webpgtmpnm

echo "</tr>" >> $filePath/$webpgtmpnm
echo "</table>" >> $filePath/$webpgtmpnm


cat $filePath/$webpgtmpnm > $webpgpath/$webpgnm
rm $filePath/$webpgtmpnm

cat $filePath/$webpgtmpnm2 > $webpgpath/$webpgnm2
rm $filePath/$webpgtmpnm2

DeleteFiles

exit 0

```
