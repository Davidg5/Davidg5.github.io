# Bike Sharing Data
Data collected and presented by David J. Gaudet

Capital One Code Challenge

5 November 2018

### Most Common Trips

![topstarttostopsc](https://user-images.githubusercontent.com/36459447/48036734-a5729180-e137-11e8-9e57-944ac3b9371a.PNG)

(dashed circle represents a round trip)

|       | Start |     |  End  | Amount of Trips |
|-------|:-----:|:---:|:-----:|:---------------:|
|   1   |  3030 | --> |  3014 |             933 |
|   2   | 3014  | --> | 3030 | 676 |
|   3   | 3031  | --> | 3005 | 609 |
|   4   | 3048  |  o  | 3048 | 569 |
|   5   | 3005  | --> | 3031 | 513 |
|   6   | 3030  | --> | 3042 | 506 |
|   7   | 3022  |  o  | 3022 | 450 |
|   8   | 3069  |  o  | 3069 | 440 |
|   9   | 3082  |  o  | 3082 | 391 |
|   10  | 3005  | --> | 3034 | 389 |

### Top Start and Stop Stations:

![startstations](https://user-images.githubusercontent.com/36459447/47973206-3468a680-e071-11e8-85d4-f548e5fd4825.PNG)
![endstations](https://user-images.githubusercontent.com/36459447/47974378-5d8c3580-e077-11e8-9794-5d97d1c00695.PNG)

Top 5 Start and Stop Stations on map:
![startandstopstationssc](https://user-images.githubusercontent.com/36459447/48035794-84a83d00-e133-11e8-9e3f-110c0d8cce42.JPG)

Green: In the top five for both start and stop (3 total)

Blue: Top five start station (1 total)

Red: Top five end station (1 total)

### Most Active Stations 
![startstopwithover5000 1](https://user-images.githubusercontent.com/36459447/48039670-deb0fe80-e143-11e8-848e-fc1279bb7a03.JPG)

The purple circles are stations with over 5000 arrivals and departures. They are the most active stations. I found it interesting that they are clustered closer towards the center of all the stations, while the less popular stations (black circles) are near the outside. I anticipated that the stations closer towards the city center would be more active, before I plotted them, so it was nice to confirm it visually.


### What is the average distance traveled? 
1.18 kilometers (round trips not included)
### How many riders include bike sharing as a regular part of their commute?
170 riders
```markdown
unique riders per day estimate = (# monthly pass trips + # flex pass trips + # staff annual trips)/(# days x typical # of trips)
                               = (81304 monthly + 9517 flex + 382 staff) / (268 days x 2 trips)
                               = 170 unique riders / day
```
### Breakdown of passholder types:

![passholdertype](https://user-images.githubusercontent.com/36459447/47974723-e8b9fb00-e078-11e8-9fbd-74b61cf4b7a4.PNG)

### Net change of bikes
One of the challenges with bike sharing is that there is not a perfect balance between the arrival of bikes and departures. This means that if there are more departures than arrivals at a station, there will be a net decrease in the amount of bikes at that station. If the opposite happens, there will be too many bikes at a station and not enough places to lock them. 

Below is a depiction of stations that have a frequent deficit in bikes (green), and stations that had a surplus in bikes (red), over the period that this data was collected.


### Data exceptions and Anomalies

Stations 3009 and 3039 are outliers in terms of location. They are both near each other, 
but relatively far away from the other stations. 

Sometimes the latitude and longitude for Station 4108 was 0,0. I decided to ignore this station.


## MATLAB Code 
Code for pass type breakdown:
```markdown
% David Gaudet
% 5 November 2018

clear;
clc;

[~, passType] = xlsread('metro-bike-share-trip-data.xlsx', 'N2:N132428')

monthly = 0;
flex = 0;
walk = 0;
staff = 0;
other = 0;
 
for i=1:132427
    if strcmp(passType(i), 'Monthly Pass')
        monthly = monthly + 1;
    elseif strcmp(passType(i), 'Flex Pass')
        flex = flex + 1;
    elseif strcmp(passType(i), 'Walk-up')
        walk = walk + 1;
    elseif strcmp(passType(i), 'Staff Annual')
        staff = staff + 1;
    else
        other = other + 1;
        fprintf('other index: %d', i);
    end
end
fprintf('Monthly: %d \n', monthly);
fprintf('Flex: %d \n', flex);
fprintf('Walk: %d \n', walk);
fprintf('Staff Annual: %d \n', staff);
fprintf('Other: %d \n', other);

% done
```
Code for calculating the average distance:
```markdown
% David J. Gaudet
% 5 November 2018

clear;
clc;

startLat = xlsread('metro-bike-share-trip-data.xlsx', 'F2:F132428');
startLong = xlsread('metro-bike-share-trip-data.xlsx', 'G2:G132428');
endLat = xlsread('metro-bike-share-trip-data.xlsx', 'I2:I132428');
endLong = xlsread('metro-bike-share-trip-data.xlsx', 'J2:J132428');


dist = 0;
nonRoundTripCount = 0;
totalDistance = 0;
for i=1:132427%132427
    dX = startLat(i) - endLat(i);
    dY = startLong(i) - endLong(i);
    if dX ~= 0 && dY ~= 0 && dX < 100 && dX > -100 && dY < 100 && dY > -100
        nonRoundTripCount = nonRoundTripCount + 1;
        distance = sqrt((111*dX)^2 + (92*dY)^2); % yes, I derived the constants 
                                                 % using a more advanced
                                                 % method of calculating
                                                 % distance. 
        totalDistance = distance + totalDistance;
    end
end
averageDistance = totalDistance / nonRoundTripCount;
fprintf('Average Distance: %.9f', averageDistance);
% done
```
Code for finding popular stations, trips, and for outputting coordinates:
```markdown
% David J. Gaudet
% 5 November 2018
clear;
clc;

% Get start and end station data
rawStartStations = xlsread('metro-bike-share-trip-data.xlsx', 'E2:E132428'); 
rawEndStations = xlsread('metro-bike-share-trip-data.xlsx', 'H2:H132428');

startStations = [];
endStations = [];
startToEnd = [];
for i=1:132427%132427
    thisStartID = rawStartStations(i);
    thisEndID = rawEndStations(i);
    % Find data exceptions , just 4108?
    if (thisStartID < 3000 || thisStartID > 3100)
        fprintf('bad station ID at: %d \n', i);
    end
    if (thisStartID > 3000 && thisStartID < 3100)
        startStations = [startStations, rawStartStations(i)];
    end
    if (thisEndID > 3000 && thisEndID < 3100)
        endStations = [endStations, rawEndStations(i)];
    end
    % Add all start to end combinations to array
    if (thisStartID > 3000 && thisStartID < 3100 ...
        && thisEndID > 3000 ...
        && thisEndID < 3100) 
        shortStart = thisStartID - 3000;
        shortEnd = thisEndID - 3000;
        num = 100*shortStart + shortEnd;
        startToEnd = [startToEnd, num];
    end
end

% Get ordered list of stations without repeats
uniqueStartIDs = unique(startStations);
uniqueEndIDs = unique(endStations);
uniqueStartToEnd = unique(startToEnd);

% Create array with frequency corresponding to array of unique IDs
startFrequency = hist(startStations, uniqueStartIDs);
endFrequency = hist(endStations, uniqueEndIDs);
startToEndFrequency = hist(startToEnd, uniqueStartToEnd);

% display IDs and their frequency next to each other *****
startFreqMatrix = [uniqueStartIDs(:), startFrequency(:)]
endFreqMatrix = [uniqueEndIDs(:), endFrequency(:)]
startToEndFreqMatrix = [uniqueStartToEnd(:), startToEndFrequency(:)];

% Create bar graphs (not used in site)
bar(uniqueStartIDs, startFrequency)
bar(uniqueEndIDs, endFrequency)
bar(uniqueStartToEnd, startToEndFrequency)

% get entry and index of max value in array
[maxStartFrequency, mostPopStartStationIndex] = max(startFrequency(:)); 
[maxEndFrequency, mostPopEndStationIndex] = max(endFrequency(:));
[maxStartToEndFrequency, mostPopStartToEndIndex] = max(startToEndFrequency(:));

% Get station and trips that are most frequent
mostPopStartStation = uniqueStartIDs(mostPopStartStationIndex);
mostPopEndStation = uniqueEndIDs(mostPopEndStationIndex);
mostPopStartToEnd = uniqueStartToEnd(mostPopStartToEndIndex);

fprintf('Most popular start station: %d (frequency = %d) \n', mostPopStartStation, maxStartFrequency);
fprintf('Most popular end station: %d (frequency = %d) \n', mostPopEndStation, maxEndFrequency);

%%%%%%%%%%%%%%%%%%%


startLat = xlsread('metro-bike-share-trip-data.xlsx', 'F2:F132428');
startLong = xlsread('metro-bike-share-trip-data.xlsx', 'G2:G132428');
endLat = xlsread('metro-bike-share-trip-data.xlsx', 'I2:I132428');
endLong = xlsread('metro-bike-share-trip-data.xlsx', 'J2:J132428');

% script to output coordinates in format for plotting website
startLats = [];
startLongs = [];
for i=1:length(uniqueStartIDs)
    j = 1;
    while rawStartStations(j) ~= uniqueStartIDs(i)
        j=j+1;
    end
    xCoord = startLat(j);
    yCoord = startLong(j);
    if xCoord < 34.026 %exclude the two outlier stations, 3009 and 3039
        fprintf('station: %d , row: %d \n', rawStartStations(j), j);
    end
    
    startLats = [startLats, xCoord];
    startLongs = [startLongs, yCoord];
    if ((uniqueStartIDs(i) == 3030) ...
        || (uniqueStartIDs(i) == 3014) ...
        || (uniqueStartIDs(i) == 3031) ...
        || (uniqueStartIDs(i) == 3005) ...
        || (uniqueStartIDs(i) == 3048) ...
        || (uniqueStartIDs(i) == 3042) ...
        || (uniqueStartIDs(i) == 3022) ...
        || (uniqueStartIDs(i) == 3069) ...
        || (uniqueStartIDs(i) == 3082) ...
        || (uniqueStartIDs(i) == 3034))
        fprintf('%f, %f, %d, #000000 \n', xCoord, yCoord, uniqueStartIDs(i));
    else
        fprintf('%f, %f, %d, #ee6a6a \n', xCoord, yCoord, uniqueStartIDs(i));
    end
    
end

% end
```



## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/Davidg5/Davidg5.github.io/edit/master/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/Davidg5/Davidg5.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
