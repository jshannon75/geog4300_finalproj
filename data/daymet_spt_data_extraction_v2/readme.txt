Daymet Multiple Extraction Tool
To learn more about this tool, go to our GitHub Page:
https://github.com/ornldaac/daymet-single-pixel-batch


1. Edit latlon.txt with your desired locations. You can also specify
Daymet Years and Variables. Example latlon.txt files can be found at:
https://github.com/ornldaac/daymet-single-pixel-batch

2. Open your command line (make sure you are in the unzipped daymet_v2 directory)

3. Run one of the following commands:

$ java -jar -Xms512m -Xmx1024m -Dhttps.protocols=TLSv1.1,TLSv1.2 daymet_multiple_extraction.jar latlon.txt

OR

$ bash daymet_multiple_extraction.sh
