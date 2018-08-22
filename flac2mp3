#! /bin/bash
#
# This script converts a directory containing .flac files to MP3 using lame encoder
#
# Basic logic for FLAC --> ID3v1, v2 conversion
# If ALBUMARTIST defined then ALBUMARTIST --> (-, TPE2)
#   else if BAND defined then BAND --> (-, TPE2)
#     else ARTIST --> (-, TPE2)
# ALBUM --> (ALBUM, TALB)
# ARTIST --> (ARTIST, TPE1)
# TRACKNUMBER --> (TRACK, TRCK)
# TITLE --> (TITLE, TIT2)
# GENRE --> (GENRE, TCON)
# COMMENT --> (COMMENT, COMM)
# If DATE defined then DATE --> (YEAR, TYER)
#   else if YEAR defined then YEAR --> (YEAR, TYER)
# If DISCNUMBER defined then DISCNUMBER --> (-, TPOS)
#   else TPOS=1
# If coverfile defined, add that to MP3 as well (ID3v2 only, APIC):
#   search order for cover images: TPE2 - TALB.*, TALB.*, cover.* Cover.* 
#                                  folder.* Folder.* front.* Front.*
#                                  where extension can be jpg, png or gif
# Version history
# 1.0 first version
# 2.0 extended with command line options
# 2.1 in case of error in processing, dumps the content of lame command line
#     to help debug
#     uses mktemp instead of using fixed file names 
#     (can run multiple instances of script)

#set -xv

VERSION="2.1"

print_usage ()
{
   echo "Usage: `basename $0` [options] <src-path>"
   echo "Options: -h   Print usage"
   echo "         -d   Delete FLAC input file after processing"
   echo "         -D<dest-path>   Path to output MP3 files (default is ../Mp3)"
   echo "         -Vn  Set the target bitrate (quality) of LAME VBR encoder"
   echo "          where n = 0 target bitrate 245 kbit/s"
   echo "                n = 1 target bitrate 225 kbit/s"
   echo "                n = 2 target bitrate 190 kbit/s"
   echo "                n = 3 target bitrate 175 kbit/s"
   echo "                n = 4 target bitrate 165 kbit/s"
   echo "                n = 5 target bitrate 130 kbit/s (default)"
   echo "                n = 6 target bitrate 115 kbit/s"
   echo "                n = 7 target bitrate 100 kbit/s"
   echo "                n = 8 target bitrate 85 kbit/s"
   echo "                n = 9 target bitrate 65 kbit/s"
   echo ""
   echo "Converts all *.flac files under <src-path> to MP3 format using LAME encoder"
   echo "result files are placed to <dest-path>/<src-path>, default is ../Mp3/<src-path>"
   echo "<src-path> should be a relative path for correct operation"
}


NO_ARGS=0
E_OPTERROR=85
FLAC_DELETE=0
MP3PATH="../Mp3"
LAME_VBR=5

echo "`basename $0` [$VERSION] - FLAC to MP3 conversion script"
 
# Test if script invoked without command-line arguments
if [ $# -eq "$NO_ARGS" ]
then
   print_usage
   exit $E_OPTERROR
fi

while getopts ":dhD:V:" Option
   do
      case $Option in
         d ) FLAC_DELETE=1
             echo "FLAC source files will be deleted after conversion";;
         h ) print_usage;;
         D ) MP3PATH=$OPTARG
             echo "MP3 output files will be written directory (relative to source dir): $MP3PATH";;
         V ) if [ "$OPTARG" -ge 0 ] && [ "$OPTARG" -le 9 ]
             then
                LAME_VBR=$OPTARG
                echo "LAME VBR quality is set to $LAME_VBR"
             else
                print_usage
             fi;;
      esac
   done
shift $(($OPTIND - 1))

if [ ! -e "$MP3PATH/$1" ]
then
    CRPATH="$MP3PATH/$1"
    CRDIRSCRIPT=$(mktemp)
    echo "Creating directory: $CRPATH"
    echo "mkdir \"$CRPATH\"" >$CRDIRSCRIPT
    chmod 700 $CRDIRSCRIPT
    $CRDIRSCRIPT
    rm $CRDIRSCRIPT
fi

declare -i RESULTALL
declare -i RESULT
declare -i NUMFILES
declare -i COVEREXIST

RESULTALL=0
NUMFILES=0

for a in "$1/"*.flac; do
    echo "============================================================"
    echo "File to be processed: $a"
    OUTF=${a%.flac}.mp3
    echo "Output file name:    `basename "$OUTF"`"
    OUTF="$MP3PATH/$OUTF"

    echo "-------------------- Source tags ---------------------------"
    ALBUM=`metaflac "$a" --show-tag=ALBUM | sed s/.*=//g`
    echo "Album:                       <$ALBUM>"
    ALBUMARTIST=`metaflac "$a" --show-tag=ALBUMARTIST | sed s/.*=//g`
    if [ -n "$ALBUMARTIST" ]
    then
        echo "Album artist:                <$ALBUMARTIST>"
    fi
    ARTIST=`metaflac "$a" --show-tag=ARTIST | sed s/.*=//g`
    echo "Artist:                      <$ARTIST>"
    BAND=`metaflac "$a" --show-tag=BAND | sed s/.*=//g`
    if [ -n "$BAND" ]
    then
        echo "Band/orchesra:               <$BAND>"
    fi
    COMMENT=`metaflac "$a" --show-tag=COMMENT | sed s/.*=//g`
    if [ -n "$COMMENT" ]
    then
        echo "Comment:                     <$COMMENT>"
    fi
    COMPOSER=`metaflac "$a" --show-tag=COMPOSER | sed s/.*=//g`
    if [ -n "$COMPOSER" ]
    then
        echo "Composer:                    <$COMPOSER>"
    fi
    DATE=`metaflac "$a" --show-tag=DATE | sed s/.*=//g`
    if [ -n "$DATE" ]
    then
        echo "Date:                        <$DATE>"
    fi
    DISCNUMBER=`metaflac "$a" --show-tag=DISCNUMBER | sed s/.*=//g`
    if [ -n "$DISCNUMBER" ]
    then
        echo "Disc #:                      <$DISCNUMBER>"
    fi
    GENRE=`metaflac "$a" --show-tag=GENRE | sed s/.*=//g`
    if [ -n "$GENRE" ]
    then
        echo "Genre:                       <$GENRE>"
    fi
    TITLE=`metaflac "$a" --show-tag=TITLE | sed s/.*=//g`
    echo "Title:                       <$TITLE>"
    TOTALDISCS=`metaflac "$a" --show-tag=TOTALDISCS | sed s/.*=//g`
    if [ -n "$TOTALDISCS" ]
    then
        echo "# of Discs:                  <$TOTALDISCS>"
    fi
    TOTALTRACKS=`metaflac "$a" --show-tag=TOTALTRACKS | sed s/.*=//g`
    if [ -n "$TOTALTRACKS" ]
    then
        echo "# of Tracks:                 <$TOTALTRACKS>"
    fi
    TRACKNUMBER=`metaflac "$a" --show-tag=TRACKNUMBER | sed s/.*=//g`
    echo "Track #:                     <$TRACKNUMBER>"
    YEAR=`metaflac "$a" --show-tag=YEAR | sed s/.*=//g`
    if [ -n "$YEAR" ]
    then
        echo "Year:                        <$YEAR>"
    fi

    if [ -n "$ALBUMARTIST" ]
    then
        TPE2=$ALBUMARTIST
    else
        if [ -n "$BAND" ]
        then
            TPE2=$BAND
        else
            TPE2=$ARTIST
        fi
    fi
    if [ -n "$DATE" ]
    then
        TYER=$DATE
    else
	    if [ -n $YEAR ]
		then
            TYER=$YEAR
		fi
    fi
    TALB=$ALBUM
    TPE1=$ARTIST
    TRCK=$TRACKNUMBER
    TIT2=$TITLE
    TCON=$GENRE
    COMM=$COMMENT
    if [ -n "$DISCNUMBER" ]
    then
        TPOS=$DISCNUMBER
    else
        TPOS="1"
    fi
    if [ -n "$TOTALDISCS" ]
    then
        TPOS="$TPOS/$TOTALDISCS"
    else
        TPOS="$TPOS/1"
    fi
    COVEREXIST=0
    for img in "$TPE2 - $TALB" "$TALB" "cover" "Cover" "folder" "Folder" "front" "Front" 
    do
        for ext in "jpg" "gif" "png"
        do
            COVERFILE="`dirname "$a"`/"$img"."$ext""
            # echo "Trying: $COVERFILE"
            if [ -e "$COVERFILE" ]
            then
                COVEREXIST=1
                break
            fi
        done
        if [ $COVEREXIST -eq 1 ]
        then
            echo "Cover image file found:      <`basename "$COVERFILE"`>"
            APIC=$COVERFILE
            break
        fi
    done
    if [ $COVEREXIST -eq 0 ]
    then
        echo "Cover image file:            <-none->"
    fi

    echo "-------------------- Output tags ---------------------------"
    echo "TPOS (Disc number):          <$TPOS>"
    echo "TPE2 (Album artist/band):    <$TPE2>"
    echo "TALB (Album):                <$TALB>"
    echo "TRCK (Track number):         <$TRCK>"
    echo "TPE1 (Track artist):         <$TPE1>"
    echo "TIT2 (Track title):          <$TIT2>"
    echo "TCON (Genre):                <$TCON>"
    echo "TYER (Album date/year):      <$TYER>"
    echo "COMM (Comment):              <$COMM>"
    if [ $COVEREXIST -eq 1 ]
    then
        echo "APIC (Cover art file):       <`basename "$APIC"`>"
    else
        echo "APIC (Cover art file):       <-none->"
    fi
    echo ""

    LAMEPARAMS="--ta \"${TPE1}\" --tl \"${TALB}\" --tn \"$TRCK\" --tt \"$TIT2\" "
    if [ -n "$TPOS" ]
    then
        LAMEPARAMS="$LAMEPARAMS --tv TPOS=\"$TPOS\""
    fi
    if [ -n "$TPE2" ]
    then
        LAMEPARAMS="$LAMEPARAMS --tv TPE2=\"$TPE2\""
    fi
    if [ -n "$TCON" ]
    then
        LAMEPARAMS="$LAMEPARAMS --tg \"$TCON\""
    fi
    if [ -n "$TYER" ]
    then
        LAMEPARAMS="$LAMEPARAMS --ty \"$TYER\""
    fi
    if [ -n "$COMM" ]
    then
        LAMEPARAMS="$LAMEPARAMS --tc \"$COMM\""
    fi
    if [ -n "$APIC" ]
    then
        LAMEPARAMS="$LAMEPARAMS --ti \"$APIC\""
    fi

    LAMEPARAMS="flac -c -d \"$a\" | lame  -V$LAME_VBR --add-id3v2 --pad-id3v2 $LAMEPARAMS - \"$OUTF\""
    LAMESCRIPT=$(mktemp)
    echo "Temporary file for lame parameters: $LAMESCRIPT"
    echo $LAMEPARAMS > $LAMESCRIPT
    chmod 700 $LAMESCRIPT
    $LAMESCRIPT
    RESULT=$?
    RESULTALL=$RESULTALL+$RESULT
    if [ $RESULT = 0 ]
    then
        echo "Current operation result: OK ($RESULT)"
    else
        echo "Current operation result: *** Error *** ($RESULT)"
        cat "$LAMESCRIPT"
    fi
    if [ -e "$LAMESCRIPT" ]
    then
        rm $LAMESCRIPT
    fi
    if [ $FLAC_DELETE -eq 1 ] && [ $RESULT -eq 0 ]; then
        rm "$a"
        echo "Input file $a deleted."
    fi
    (( NUMFILES++ ))
done
echo "============================================================"
echo "Conversion complete"
echo "Files processed: $NUMFILES"
if [ $RESULTALL = 0 ]
then
    echo "Conversion process result: OK ($RESULTALL)"
else
    echo "Conversion process result: *** Error *** ($RESULTALL)"
fi

exit $RESULTALL
