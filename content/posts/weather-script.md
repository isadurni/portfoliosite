+++
authors = ["Ignacio Sadurni"]
title = "Live Weather Forecast"
date = "2024-02-10"
description = "A brief guide to setup KaTeX"
tags = [
    "Systems Programming",
    "Shell Script",
]
categories = []
series = []
+++

Implemented Shell commands in Linux for generating live weather forecasts. Enabled user input of zip codes to fetch real-time data and display current conditions. Provided concise forecast summaries for enhanced user experience.

---

**NOTE.** For more information on how the program works, read the provided text and code below:

{{< notice info >}}
**How does it work?**

1. The program is executed in linux and displays the weather information. Inputting flags as arguments, the user can change between farenheit and celcius. Additionally, they can include a zipcode, otherwise it defaults to South Bend, IN.

2. I created an alias and implemented the shell script so that it displays the weather forecast information for South Bend, every time I login into my Linux Terminal.

{{< /notice >}}

**COMMAND LINE OUTPUT.** The following is a sample of what the ouput in command line looks like:

{{< highlight html >}}

./weather.sh -f
Temperature: 55 degrees
Forecast:    Fair

{{< /highlight >}}


**SOURCE CODE.** The following is a file written in shell script:

{{< highlight shell >}}

#!/bin/sh

# Globals

URL="https://forecast.weather.gov/zipcity.php"
ZIPCODE=46556
FORECAST=0
CELSIUS=0

# Functions

usage() {
    cat 1>&2 <<EOF
Usage: $(basename $0) [zipcode]

-c    Use Celsius degrees instead of Fahrenheit for temperature
-f    Display forecast text after the temperature

If zipcode is not provided, then it defaults to $ZIPCODE.
EOF
    exit $1
}

weather_information() {
    # Fetch weather information from URL based on ZIPCODEi
	curl -sL https://forecast.weather.gov/zipcity.php?inputstring=$ZIPCODE
}

temperature() {
    # Extract temperature information from weather source
	if [ "$CELSIUS" -eq 1 ]; then
   		weather_information | grep "myforecast-current-s" | cut -d '>' -f2 | grep -Eo '[0-9]+'
	else 
		weather_information | grep "myforecast-current-l" | cut -d '>' -f2 | grep -Eo '[0-9]+'
	fi
}

forecast() {
    # Extract forecast information from weather source
    weather_information | grep "myforecast-current[^-]" | cut -d '>' -f2 | cut -d '<' -f1 | sed 's/^[ \t]*//;s/[ \t]*$//'
}

# Parse Command Line Options

while [ $# -gt 0 ]; do
    case $1 in
	-h) usage 0;;
	-c) CELSIUS=1;;
	-f) FORECAST=1;;
	*) ZIPCODE=$1;;
    esac
    shift
done

# Display Information

echo "Temperature: $(temperature) degrees"

if [ "$FORECAST" -eq 1 ]; then
	echo "Forecast:    $(forecast)"
fi

{{< /highlight >}}