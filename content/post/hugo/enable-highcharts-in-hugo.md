---
title: "在Hugo中开启Highcharts图表支持"
date: 2023-04-02T16:39:51+08:00
lastmod: 2023-04-03T16:39:51+08:00
draft: true
keywords: ["hugo","highcharts","go"]
description: "简要介绍如何在Hugo中开启Highcharts图表支持"
tags: ["hugo","highcharts","go"]
categories: ["个人博客","系统集成"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: true
  options: "
   {
    subtitle: {
        style: {
            color: 'red'
        }
    }
  }
"
---

简要说明如何在`Hugo`中集成[Highcharts](https://www.highcharts.com/)。

<!--more-->

# 修改过程

# 展示效果

## 图表1-Bubble chart

```highcharts
  {
  
      chart: {
          type: 'bubble',
          plotBorderWidth: 1,
          zoomType: 'xy'
      },
  
      legend: {
          enabled: false
      },
  
      title: {
          text: 'Sugar and fat intake per country'
      },
  
      subtitle: {
          text: 'Source: <a href="http://www.euromonitor.com/">Euromonitor</a> and <a href="https://data.oecd.org/">OECD</a>'
      },
  
      accessibility: {
          point: {
              valueDescriptionFormat: '{index}. {point.name}, fat: {point.x}g, sugar: {point.y}g, obesity: {point.z}%.'
          }
      },
  
      xAxis: {
          gridLineWidth: 1,
          title: {
              text: 'Daily fat intake'
          },
          labels: {
              format: '{value} gr'
          },
          plotLines: [{
              color: 'black',
              dashStyle: 'dot',
              width: 2,
              value: 65,
              label: {
                  rotation: 0,
                  y: 15,
                  style: {
                      fontStyle: 'italic'
                  },
                  text: 'Safe fat intake 65g/day'
              },
              zIndex: 3
          }],
          accessibility: {
              rangeDescription: 'Range: 60 to 100 grams.'
          }
      },
  
      yAxis: {
          startOnTick: false,
          endOnTick: false,
          title: {
              text: 'Daily sugar intake'
          },
          labels: {
              format: '{value} gr'
          },
          maxPadding: 0.2,
          plotLines: [{
              color: 'black',
              dashStyle: 'dot',
              width: 2,
              value: 50,
              label: {
                  align: 'right',
                  style: {
                      fontStyle: 'italic'
                  },
                  text: 'Safe sugar intake 50g/day',
                  x: -10
              },
              zIndex: 3
          }],
          accessibility: {
              rangeDescription: 'Range: 0 to 160 grams.'
          }
      },
  
      tooltip: {
          useHTML: true,
          headerFormat: '<table>',
          pointFormat: '<tr><th colspan="2"><h3>{point.country}</h3></th></tr>' +
              '<tr><th>Fat intake:</th><td>{point.x}g</td></tr>' +
              '<tr><th>Sugar intake:</th><td>{point.y}g</td></tr>' +
              '<tr><th>Obesity (adults):</th><td>{point.z}%</td></tr>',
          footerFormat: '</table>',
          followPointer: true
      },
  
      plotOptions: {
          series: {
              dataLabels: {
                  enabled: true,
                  format: '{point.name}'
              }
          }
      },
  
      series: [{
          data: [
              { x: 95, y: 95, z: 13.8, name: 'BE', country: 'Belgium' },
              { x: 86.5, y: 102.9, z: 14.7, name: 'DE', country: 'Germany' },
              { x: 80.8, y: 91.5, z: 15.8, name: 'FI', country: 'Finland' },
              { x: 80.4, y: 102.5, z: 12, name: 'NL', country: 'Netherlands' },
              { x: 80.3, y: 86.1, z: 11.8, name: 'SE', country: 'Sweden' },
              { x: 78.4, y: 70.1, z: 16.6, name: 'ES', country: 'Spain' },
              { x: 74.2, y: 68.5, z: 14.5, name: 'FR', country: 'France' },
              { x: 73.5, y: 83.1, z: 10, name: 'NO', country: 'Norway' },
              { x: 71, y: 93.2, z: 24.7, name: 'UK', country: 'United Kingdom' },
              { x: 69.2, y: 57.6, z: 10.4, name: 'IT', country: 'Italy' },
              { x: 68.6, y: 20, z: 16, name: 'RU', country: 'Russia' },
              { x: 65.5, y: 126.4, z: 35.3, name: 'US', country: 'United States' },
              { x: 65.4, y: 50.8, z: 28.5, name: 'HU', country: 'Hungary' },
              { x: 63.4, y: 51.8, z: 15.4, name: 'PT', country: 'Portugal' },
              { x: 64, y: 82.9, z: 31.3, name: 'NZ', country: 'New Zealand' }
          ],
          colorByPoint: true
      }]
  
  }
```

## 图表2-Basic column

```highcharts
{
      chart: {
          type: 'column'
      },
      title: {
          text: 'Monthly Average Rainfall'
      },
      subtitle: {
          text: 'Source: WorldClimate.com'
      },
      xAxis: {
          categories: [
              'Jan',
              'Feb',
              'Mar',
              'Apr',
              'May',
              'Jun',
              'Jul',
              'Aug',
              'Sep',
              'Oct',
              'Nov',
              'Dec'
          ],
          crosshair: true
      },
      yAxis: {
          min: 0,
          title: {
              text: 'Rainfall (mm)'
          }
      },
      tooltip: {
          headerFormat: '<span style="font-size:10px">{point.key}</span><table>',
          pointFormat: '<tr><td style="color:{series.color};padding:0">{series.name}: </td>' +
              '<td style="padding:0"><b>{point.y:.1f} mm</b></td></tr>',
          footerFormat: '</table>',
          shared: true,
          useHTML: true
      },
      plotOptions: {
          column: {
              pointPadding: 0.2,
              borderWidth: 0
          }
      },
      series: [{
          name: 'Tokyo',
          data: [49.9, 71.5, 106.4, 129.2, 144.0, 176.0, 135.6, 148.5, 216.4,
              194.1, 95.6, 54.4]
  
      }, {
          name: 'New York',
          data: [83.6, 78.8, 98.5, 93.4, 106.0, 84.5, 105.0, 104.3, 91.2, 83.5,
              106.6, 92.3]
  
      }, {
          name: 'London',
          data: [48.9, 38.8, 39.3, 41.4, 47.0, 48.3, 59.0, 59.6, 52.4, 65.2, 59.3,
              51.2]
  
      }, {
          name: 'Berlin',
          data: [42.4, 33.2, 34.5, 39.7, 52.6, 75.5, 57.4, 60.4, 47.6, 39.1, 46.8,
              51.1]
  
      }]
  }
```

## 图表3-Basic line

```highcharts
  {
  
      title: {
          text: 'U.S Solar Employment Growth by Job Category, 2010-2020',
          align: 'left'
      },
  
      subtitle: {
          text: 'Source: <a href="https://irecusa.org/programs/solar-jobs-census/" target="_blank">IREC</a>',
          align: 'left'
      },
  
      yAxis: {
          title: {
              text: 'Number of Employees'
          }
      },
  
      xAxis: {
          accessibility: {
              rangeDescription: 'Range: 2010 to 2020'
          }
      },
  
      legend: {
          layout: 'vertical',
          align: 'right',
          verticalAlign: 'middle'
      },
  
      plotOptions: {
          series: {
              label: {
                  connectorAllowed: false
              },
              pointStart: 2010
          }
      },
  
      series: [{
          name: 'Installation & Developers',
          data: [43934, 48656, 65165, 81827, 112143, 142383,
              171533, 165174, 155157, 161454, 154610]
      }, {
          name: 'Manufacturing',
          data: [24916, 37941, 29742, 29851, 32490, 30282,
              38121, 36885, 33726, 34243, 31050]
      }, {
          name: 'Sales & Distribution',
          data: [11744, 30000, 16005, 19771, 20185, 24377,
              32147, 30912, 29243, 29213, 25663]
      }, {
          name: 'Operations & Maintenance',
          data: [null, null, null, null, null, null, null,
              null, 11164, 11218, 10077]
      }, {
          name: 'Other',
          data: [21908, 5548, 8105, 11248, 8989, 11816, 18274,
              17300, 13053, 11906, 10073]
      }],
  
      responsive: {
          rules: [{
              condition: {
                  maxWidth: 500
              },
              chartOptions: {
                  legend: {
                      layout: 'horizontal',
                      align: 'center',
                      verticalAlign: 'bottom'
                  }
              }
          }]
      }
  }
```

## 图表4-3D donut

```highcharts
 {
      chart: {
          type: 'pie',
          options3d: {
              enabled: true,
              alpha: 45
          }
      },
      title: {
          text: 'Beijing 2022 gold medals by country',
          align: 'left'
      },
      subtitle: {
          text: '3D donut in Highcharts',
          align: 'left'
      },
      plotOptions: {
          pie: {
              innerSize: 100,
              depth: 45
          }
      },
      series: [{
          name: 'Medals',
          data: [
              ['Norway', 16],
              ['Germany', 12],
              ['USA', 8],
              ['Sweden', 8],
              ['Netherlands', 8],
              ['ROC', 6],
              ['Austria', 7],
              ['Canada', 4],
              ['Japan', 3]
  
          ]
      }]
  }
```

## 图表5-Tile map

```highcharts
  {
      chart: {
          type: 'tilemap',
          inverted: true,
          height: '80%'
      },
  
      accessibility: {
          description: 'A tile map represents the states of the USA by population in 2016. The hexagonal tiles are positioned to geographically echo the map of the USA. A color-coded legend states the population levels as below 1 million (beige), 1 to 5 million (orange), 5 to 20 million (pink) and above 20 million (hot pink). The chart is interactive, and the individual state data points are displayed upon hovering. Three states have a population of above 20 million: California (39.3 million), Texas (27.9 million) and Florida (20.6 million). The northern US region from Massachusetts in the Northwest to Illinois in the Midwest contains the highest concentration of states with a population of 5 to 20 million people. The southern US region from South Carolina in the Southeast to New Mexico in the Southwest contains the highest concentration of states with a population of 1 to 5 million people. 6 states have a population of less than 1 million people; these include Alaska, Delaware, Wyoming, North Dakota, South Dakota and Vermont. The state with the lowest population is Wyoming in the Northwest with 584,153 people.',
          screenReaderSection: {
              beforeChartFormat:
                  '<h5>{chartTitle}</h5>' +
                  '<div>{chartSubtitle}</div>' +
                  '<div>{chartLongdesc}</div>' +
                  '<div>{viewTableButton}</div>'
          },
          point: {
              valueDescriptionFormat: '{index}. {xDescription}, {point.value}.'
          }
      },
  
      title: {
          text: 'U.S. states by population in 2016'
      },
  
      subtitle: {
          text: 'Source:<a href="https://simple.wikipedia.org/wiki/List_of_U.S._states_by_population">Wikipedia</a>'
      },
  
      xAxis: {
          visible: false
      },
  
      yAxis: {
          visible: false
      },
  
      colorAxis: {
          dataClasses: [{
              from: 0,
              to: 1000000,
              color: '#F9EDB3',
              name: '< 1M'
          }, {
              from: 1000000,
              to: 5000000,
              color: '#FFC428',
              name: '1M - 5M'
          }, {
              from: 5000000,
              to: 20000000,
              color: '#FF7987',
              name: '5M - 20M'
          }, {
              from: 20000000,
              color: '#FF2371',
              name: '> 20M'
          }]
      },
  
      tooltip: {
          headerFormat: '',
          pointFormat: 'The population of <b> {point.name}</b> is <b>{point.value}</b>'
      },
  
      plotOptions: {
          series: {
              dataLabels: {
                  enabled: true,
                  format: '{point.hc-a2}',
                  color: '#000000',
                  style: {
                      textOutline: false
                  }
              }
          }
      },
  
      series: [{
          name: '',
          data: [{
              'hc-a2': 'AL',
              name: 'Alabama',
              region: 'South',
              x: 6,
              y: 7,
              value: 4849377
          }, {
              'hc-a2': 'AK',
              name: 'Alaska',
              region: 'West',
              x: 0,
              y: 0,
              value: 737732
          }, {
              'hc-a2': 'AZ',
              name: 'Arizona',
              region: 'West',
              x: 5,
              y: 3,
              value: 6745408
          }, {
              'hc-a2': 'AR',
              name: 'Arkansas',
              region: 'South',
              x: 5,
              y: 6,
              value: 2994079
          }, {
              'hc-a2': 'CA',
              name: 'California',
              region: 'West',
              x: 5,
              y: 2,
              value: 39250017
          }, {
              'hc-a2': 'CO',
              name: 'Colorado',
              region: 'West',
              x: 4,
              y: 3,
              value: 5540545
          }, {
              'hc-a2': 'CT',
              name: 'Connecticut',
              region: 'Northeast',
              x: 3,
              y: 11,
              value: 3596677
          }, {
              'hc-a2': 'DE',
              name: 'Delaware',
              region: 'South',
              x: 4,
              y: 9,
              value: 935614
          }, {
              'hc-a2': 'DC',
              name: 'District of Columbia',
              region: 'South',
              x: 4,
              y: 10,
              value: 7288000
          }, {
              'hc-a2': 'FL',
              name: 'Florida',
              region: 'South',
              x: 8,
              y: 8,
              value: 20612439
          }, {
              'hc-a2': 'GA',
              name: 'Georgia',
              region: 'South',
              x: 7,
              y: 8,
              value: 10310371
          }, {
              'hc-a2': 'HI',
              name: 'Hawaii',
              region: 'West',
              x: 8,
              y: 0,
              value: 1419561
          }, {
              'hc-a2': 'ID',
              name: 'Idaho',
              region: 'West',
              x: 3,
              y: 2,
              value: 1634464
          }, {
              'hc-a2': 'IL',
              name: 'Illinois',
              region: 'Midwest',
              x: 3,
              y: 6,
              value: 12801539
          }, {
              'hc-a2': 'IN',
              name: 'Indiana',
              region: 'Midwest',
              x: 3,
              y: 7,
              value: 6596855
          }, {
              'hc-a2': 'IA',
              name: 'Iowa',
              region: 'Midwest',
              x: 3,
              y: 5,
              value: 3107126
          }, {
              'hc-a2': 'KS',
              name: 'Kansas',
              region: 'Midwest',
              x: 5,
              y: 5,
              value: 2904021
          }, {
              'hc-a2': 'KY',
              name: 'Kentucky',
              region: 'South',
              x: 4,
              y: 6,
              value: 4413457
          }, {
              'hc-a2': 'LA',
              name: 'Louisiana',
              region: 'South',
              x: 6,
              y: 5,
              value: 4649676
          }, {
              'hc-a2': 'ME',
              name: 'Maine',
              region: 'Northeast',
              x: 0,
              y: 11,
              value: 1330089
          }, {
              'hc-a2': 'MD',
              name: 'Maryland',
              region: 'South',
              x: 4,
              y: 8,
              value: 6016447
          }, {
              'hc-a2': 'MA',
              name: 'Massachusetts',
              region: 'Northeast',
              x: 2,
              y: 10,
              value: 6811779
          }, {
              'hc-a2': 'MI',
              name: 'Michigan',
              region: 'Midwest',
              x: 2,
              y: 7,
              value: 9928301
          }, {
              'hc-a2': 'MN',
              name: 'Minnesota',
              region: 'Midwest',
              x: 2,
              y: 4,
              value: 5519952
          }, {
              'hc-a2': 'MS',
              name: 'Mississippi',
              region: 'South',
              x: 6,
              y: 6,
              value: 2984926
          }, {
              'hc-a2': 'MO',
              name: 'Missouri',
              region: 'Midwest',
              x: 4,
              y: 5,
              value: 6093000
          }, {
              'hc-a2': 'MT',
              name: 'Montana',
              region: 'West',
              x: 2,
              y: 2,
              value: 1023579
          }, {
              'hc-a2': 'NE',
              name: 'Nebraska',
              region: 'Midwest',
              x: 4,
              y: 4,
              value: 1881503
          }, {
              'hc-a2': 'NV',
              name: 'Nevada',
              region: 'West',
              x: 4,
              y: 2,
              value: 2839099
          }, {
              'hc-a2': 'NH',
              name: 'New Hampshire',
              region: 'Northeast',
              x: 1,
              y: 11,
              value: 1326813
          }, {
              'hc-a2': 'NJ',
              name: 'New Jersey',
              region: 'Northeast',
              x: 3,
              y: 10,
              value: 8944469
          }, {
              'hc-a2': 'NM',
              name: 'New Mexico',
              region: 'West',
              x: 6,
              y: 3,
              value: 2085572
          }, {
              'hc-a2': 'NY',
              name: 'New York',
              region: 'Northeast',
              x: 2,
              y: 9,
              value: 19745289
          }, {
              'hc-a2': 'NC',
              name: 'North Carolina',
              region: 'South',
              x: 5,
              y: 9,
              value: 10146788
          }, {
              'hc-a2': 'ND',
              name: 'North Dakota',
              region: 'Midwest',
              x: 2,
              y: 3,
              value: 739482
          }, {
              'hc-a2': 'OH',
              name: 'Ohio',
              region: 'Midwest',
              x: 3,
              y: 8,
              value: 11614373
          }, {
              'hc-a2': 'OK',
              name: 'Oklahoma',
              region: 'South',
              x: 6,
              y: 4,
              value: 3878051
          }, {
              'hc-a2': 'OR',
              name: 'Oregon',
              region: 'West',
              x: 4,
              y: 1,
              value: 3970239
          }, {
              'hc-a2': 'PA',
              name: 'Pennsylvania',
              region: 'Northeast',
              x: 3,
              y: 9,
              value: 12784227
          }, {
              'hc-a2': 'RI',
              name: 'Rhode Island',
              region: 'Northeast',
              x: 2,
              y: 11,
              value: 1055173
          }, {
              'hc-a2': 'SC',
              name: 'South Carolina',
              region: 'South',
              x: 6,
              y: 8,
              value: 4832482
          }, {
              'hc-a2': 'SD',
              name: 'South Dakota',
              region: 'Midwest',
              x: 3,
              y: 4,
              value: 853175
          }, {
              'hc-a2': 'TN',
              name: 'Tennessee',
              region: 'South',
              x: 5,
              y: 7,
              value: 6651194
          }, {
              'hc-a2': 'TX',
              name: 'Texas',
              region: 'South',
              x: 7,
              y: 4,
              value: 27862596
          }, {
              'hc-a2': 'UT',
              name: 'Utah',
              region: 'West',
              x: 5,
              y: 4,
              value: 2942902
          }, {
              'hc-a2': 'VT',
              name: 'Vermont',
              region: 'Northeast',
              x: 1,
              y: 10,
              value: 626011
          }, {
              'hc-a2': 'VA',
              name: 'Virginia',
              region: 'South',
              x: 5,
              y: 8,
              value: 8411808
          }, {
              'hc-a2': 'WA',
              name: 'Washington',
              region: 'West',
              x: 2,
              y: 1,
              value: 7288000
          }, {
              'hc-a2': 'WV',
              name: 'West Virginia',
              region: 'South',
              x: 4,
              y: 7,
              value: 1850326
          }, {
              'hc-a2': 'WI',
              name: 'Wisconsin',
              region: 'Midwest',
              x: 2,
              y: 5,
              value: 5778708
          }, {
              'hc-a2': 'WY',
              name: 'Wyoming',
              region: 'West',
              x: 3,
              y: 3,
              value: 584153
          }]
      }]
  }
```

## 图表6-Sankey diagram

```highcharts
{
  
      title: {
          text: 'Highcharts Sankey Diagram'
      },
      accessibility: {
          point: {
              valueDescriptionFormat: '{index}. {point.from} to {point.to}, {point.weight}.'
          }
      },
      series: [{
          keys: ['from', 'to', 'weight'],
          data: [
              ['Brazil', 'Portugal', 5],
              ['Brazil', 'France', 1],
              ['Brazil', 'Spain', 1],
              ['Brazil', 'England', 1],
              ['Canada', 'Portugal', 1],
              ['Canada', 'France', 5],
              ['Canada', 'England', 1],
              ['Mexico', 'Portugal', 1],
              ['Mexico', 'France', 1],
              ['Mexico', 'Spain', 5],
              ['Mexico', 'England', 1],
              ['USA', 'Portugal', 1],
              ['USA', 'France', 1],
              ['USA', 'Spain', 1],
              ['USA', 'England', 5],
              ['Portugal', 'Angola', 2],
              ['Portugal', 'Senegal', 1],
              ['Portugal', 'Morocco', 1],
              ['Portugal', 'South Africa', 3],
              ['France', 'Angola', 1],
              ['France', 'Senegal', 3],
              ['France', 'Mali', 3],
              ['France', 'Morocco', 3],
              ['France', 'South Africa', 1],
              ['Spain', 'Senegal', 1],
              ['Spain', 'Morocco', 3],
              ['Spain', 'South Africa', 1],
              ['England', 'Angola', 1],
              ['England', 'Senegal', 1],
              ['England', 'Morocco', 2],
              ['England', 'South Africa', 7],
              ['South Africa', 'China', 5],
              ['South Africa', 'India', 1],
              ['South Africa', 'Japan', 3],
              ['Angola', 'China', 5],
              ['Angola', 'India', 1],
              ['Angola', 'Japan', 3],
              ['Senegal', 'China', 5],
              ['Senegal', 'India', 1],
              ['Senegal', 'Japan', 3],
              ['Mali', 'China', 5],
              ['Mali', 'India', 1],
              ['Mali', 'Japan', 3],
              ['Morocco', 'China', 5],
              ['Morocco', 'India', 1],
              ['Morocco', 'Japan', 3]
          ],
          type: 'sankey',
          name: 'Sankey demo series'
      }]
  
  }
```

## 图表7-Gauge series

```highcharts
{

    chart: {
        type: 'gauge',
        plotBackgroundColor: null,
        plotBackgroundImage: null,
        plotBorderWidth: 0,
        plotShadow: false,
        height: '80%'
    },

    title: {
        text: 'Speedometer'
    },

    pane: {
        startAngle: -90,
        endAngle: 89.9,
        background: null,
        center: ['50%', '75%'],
        size: '110%'
    },


    yAxis: {
        min: 0,
        max: 200,
        tickPixelInterval: 72,
        tickPosition: 'inside',
        tickColor: Highcharts.defaultOptions.chart.backgroundColor || '#FFFFFF',
        tickLength: 20,
        tickWidth: 2,
        minorTickInterval: null,
        labels: {
            distance: 20,
            style: {
                fontSize: '14px'
            }
        },
        plotBands: [{
            from: 0,
            to: 120,
            color: '#55BF3B', 
            thickness: 20
        }, {
            from: 120,
            to: 160,
            color: '#DDDF0D',
            thickness: 20
        }, {
            from: 160,
            to: 200,
            color: '#DF5353',
            thickness: 20
        }]
    },

    series: [{
        name: 'Speed',
        data: [80],
        tooltip: {
            valueSuffix: ' km/h'
        },
        dataLabels: {
            format: '{y} km/h',
            borderWidth: 0,
            color: (
                Highcharts.defaultOptions.title &&
                Highcharts.defaultOptions.title.style &&
                Highcharts.defaultOptions.title.style.color
            ) || '#333333',
            style: {
                fontSize: '16px'
            }
        },
        dial: {
            radius: '80%',
            backgroundColor: 'gray',
            baseWidth: 12,
            baseLength: '0%',
            rearLength: '0%'
        },
        pivot: {
            backgroundColor: 'gray',
            radius: 6
        }

    }]

}
```

# 特殊图表展示



