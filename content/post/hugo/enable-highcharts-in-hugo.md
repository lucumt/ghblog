---
title: "在Hugo中开启Highcharts图表支持"
date: 2023-04-02T16:39:51+08:00
lastmod: 2023-04-03T16:39:51+08:00
draft: false
keywords: ["hugo","highcharts","go"]
description: "简要介绍如何在Hugo中开启Highcharts图表支持以便支持数据统计分析相关的功能"
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

基于[在Hugo中开启图表支持](/post/hugo/enable-diagrams-in-hugo/)一文，简要说明如何在`Hugo`中基于`Markdown`以优雅的方式集成[Highcharts](https://www.highcharts.com/)。

<!--more-->

## 修改过程

1. 在`assets/sass/_custom_custom.scss`中添加如下代码

   ```scss
   .highcharts {
      border: 1px dashed #c9c9c9;
   }
   ```

2. 在`layouts/_default/_markup`创建文件`render-codeblock-highcharts.html`并添加如下代码

   ```html
   <div id="highcharts_{{ .Ordinal }}" class="highcharts">
       {{- .Inner }}
   </div>
   ```

3. 在`layouts/partials/scripts.html`补充原有的代码，添加上初始化功能

   ```html
   <!-- highcharts js -->
   {{- if and (or .Params.highchartsDiagrams.enable (and .Site.Params.highchartsDiagrams.enable (ne .Params.highchartsDiagrams.enable false))) (or .IsPage .IsHome) -}}
     <script src="https://code.highcharts.com/highcharts.js"></script>
     <script src="https://code.highcharts.com/highcharts-more.js"></script>
     <script src="https://code.highcharts.com/highcharts-3d.js"></script>
     <script src="https://code.highcharts.com/modules/heatmap.js"></script>
     <script src="https://code.highcharts.com/modules/tilemap.js"></script>
     <script src="https://code.highcharts.com/modules/sankey.js"></script>
     <!-- depends on your requirements,needs to add more extra js file -->
     <script type="text/javascript">
   	  document.addEventListener('DOMContentLoaded', initHighcharts);
   	  function initHighcharts(){
   		let highchartsPageOptions = {{ .Page.Params.highchartsDiagrams.options }};
   		let highchartsSiteOptions = {{ .Site.Params.highchartsDiagrams.options }};
   		highchartsPageOptions = !!highchartsPageOptions ? highchartsPageOptions : "{}"
   		highchartsSiteOptions = !!highchartsSiteOptions ? highchartsSiteOptions : "{}"
   
   		highchartsPageOptions = eval("(" + highchartsPageOptions + ")")
   		highchartsSiteOptions = eval("(" + highchartsSiteOptions + ")")
   		// page options have high priority then site options
   		let highchartsOptions = {...highchartsSiteOptions, ...highchartsPageOptions}; 
   		let highchartsConfigs = document.querySelectorAll("[id^=highcharts_]");
   		console.log(highchartsOptions)
   		for(config of highchartsConfigs){
   		  let chartData = eval("(" + config.innerText + ")");
   		  chartData = {...highchartsOptions,...chartData}
   		  Highcharts.chart(config.id,chartData);
   		}
   	  }
     </script>
   {{- end }}
   ```

4. 在对应的`markdown`页面头部开启`highcharts`的展示，可根据实际情况添加自定义配置

   ```javascript
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
   ```

5. 仿照如下的代码，将`Highcharts`的代码块以`JavaScript`对象的形式加入特定`Markdown`代码标签中，注意 **`JavaScript`代码中不要加入任何注释，否则会导致程序解析出错!**

   ```javascript
   ​```highcharts
    {
       chart: {
           plotBackgroundColor: null,
           plotBorderWidth: null,
           plotShadow: false,
           type: 'pie'
       },
       title: {
           text: 'Browser market shares in May, 2020',
           align: 'left'
       },
       tooltip: {
           pointFormat: '{series.name}: <b>{point.percentage:.1f}%</b>'
       },
       accessibility: {
           point: {
               valueSuffix: '%'
           }
       },
       plotOptions: {
           pie: {
               allowPointSelect: true,
               cursor: 'pointer',
               dataLabels: {
                   enabled: true,
                   format: '<b>{point.name}</b>: {point.percentage:.1f} %'
               }
           }
       },
       series: [{
           name: 'Brands',
           colorByPoint: true,
           data: [{
               name: 'Chrome',
               y: 70.67,
               sliced: true,
               selected: true
           }, {
               name: 'Edge',
               y: 14.77
           },  {
               name: 'Firefox',
               y: 4.86
           }, {
               name: 'Safari',
               y: 2.63
           }, {
               name: 'Internet Explorer',
               y: 1.53
           },  {
               name: 'Opera',
               y: 1.40
           }, {
               name: 'Sogou Explorer',
               y: 0.84
           }, {
               name: 'QQ',
               y: 0.51
           }, {
               name: 'Other',
               y: 2.6
           }]
       }]
   }
   ​```
   ```

   让`Hugo`重新选然后即可展示出类似如下效果

   ```highcharts
    {
       chart: {
           plotBackgroundColor: null,
           plotBorderWidth: null,
           plotShadow: false,
           type: 'pie'
       },
       title: {
           text: 'Browser market shares in May, 2020',
           align: 'left'
       },
       tooltip: {
           pointFormat: '{series.name}: <b>{point.percentage:.1f}%</b>'
       },
       accessibility: {
           point: {
               valueSuffix: '%'
           }
       },
       plotOptions: {
           pie: {
               allowPointSelect: true,
               cursor: 'pointer',
               dataLabels: {
                   enabled: true,
                   format: '<b>{point.name}</b>: {point.percentage:.1f} %'
               }
           }
       },
       series: [{
           name: 'Brands',
           colorByPoint: true,
           data: [{
               name: 'Chrome',
               y: 70.67,
               sliced: true,
               selected: true
           }, {
               name: 'Edge',
               y: 14.77
           },  {
               name: 'Firefox',
               y: 4.86
           }, {
               name: 'Safari',
               y: 2.63
           }, {
               name: 'Internet Explorer',
               y: 1.53
           },  {
               name: 'Opera',
               y: 1.40
           }, {
               name: 'Sogou Explorer',
               y: 0.84
           }, {
               name: 'QQ',
               y: 0.51
           }, {
               name: 'Other',
               y: 2.6
           }]
       }]
   }
   ```

## 展示效果

处于篇幅考虑，在展示部分只展示最终效果，原始代码请参见[enable-highcharts-in-hugo.md](https://raw.githubusercontent.com/lucumt/ghblog/master/content/post/hugo/enable-highcharts-in-hugo.md)

### 图1-Bubble chart

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

### 图2-Basic column

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

### 图3-Basic line

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

### 图4-3D donut

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

### 图5-Tile map

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

### 图6-Sankey diagram

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

### 图7-Gauge series

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

### 图8-Arc diagram

```highcharts
 {

    colors: ['#293462', '#a64942', '#fe5f55', '#fff1c1', '#5bd1d7', '#ff502f', '#004d61', '#ff8a5c', '#fff591', '#f5587b', '#fad3cf', '#a696c8', '#5BE7C4', '#266A2E', '#593E1A'],

    title: {
        text: 'Main train connections in Europe'
    },

    accessibility: {
        description: 'Arc diagram chart with circles of different sizes along the X axis, and connections drawn as arcs between them. From the chart we can see that Paris is the city with the most connections to other cities.',
        point: {
            valueDescriptionFormat: 'Connection from {point.from} to {point.to}.'
        }
    },

    series: [{
        keys: ['from', 'to', 'weight'],
        type: 'arcdiagram',
        name: 'Train connections',
        linkWeight: 1,
        centeredLinks: true,
        dataLabels: {
            rotation: 90,
            y: 30,
            align: 'left',
            color: 'black'
        },
        offset: '65%',
        data: [
            ['Hamburg', 'Stuttgart', 1],
            ['Hamburg', 'Frankfurt', 1],
            ['Hamburg', 'München', 1],
            ['Hannover', 'Wien', 1],
            ['Hannover', 'München', 1],
            ['Berlin', 'Wien', 1],
            ['Berlin', 'München', 1],
            ['Berlin', 'Stuttgart', 1],
            ['Berlin', 'Frankfurt', 1],
            ['Berlin', 'Köln', 1],
            ['Berlin', 'Düsseldorf', 1],
            ['München', 'Düsseldorf', 1],
            ['München', 'Wien', 1],
            ['München', 'Frankfurt', 1],
            ['München', 'Köln', 1],
            ['München', 'Amsterdam', 1],
            ['Stuttgart', 'Wien', 1],
            ['Frankfurt', 'Wien', 1],
            ['Frankfurt', 'Amsterdam', 1],
            ['Frankfurt', 'Paris', 1],
            ['Frankfurt', 'Budapest', 1],
            ['Düsseldorf', 'Wien', 1],
            ['Düsseldorf', 'Hamburg', 1],
            ['Amsterdam', 'Paris', 1],
            ['Paris', 'Brest', 1],
            ['Paris', 'Nantes', 1],
            ['Paris', 'Bayonne', 1],
            ['Paris', 'Bordeaux', 1],
            ['Paris', 'Toulouse', 1],
            ['Paris', 'Montpellier', 1],
            ['Paris', 'Marseille', 1],
            ['Paris', 'Nice', 1],
            ['Paris', 'Milano', 1],
            ['Nantes', 'Nice', 1],
            ['Bordeaux', 'Lyon', 1],
            ['Nantes', 'Lyon', 1],
            ['Milano', 'München', 1],
            ['Milano', 'Roma', 1],
            ['Milano', 'Bari', 1],
            ['Milano', 'Napoli', 1],
            ['Milano', 'Brindisi', 1],
            ['Milano', 'Lamezia Terme', 1],
            ['Torino', 'Roma', 1],
            ['Venezia', 'Napoli', 1],
            ['Roma', 'Bari', 1],
            ['Roma', 'Catania', 1],
            ['Roma', 'Brindisi', 1],
            ['Catania', 'Milano', 1]
        ]
    }]

}
```

## 特殊图表

对于某些复杂的`Highcharts`图表，可尝试将其转化为一个`JavaScript`对象以兼容前述的实现方案，保持优雅。若实现难度较大，可将相关的`html`代码直接嵌入`markdown`中来实现相关功能，但通过此种方式会导致无法对相关的图表进行动态配置。

下述为一个示例展示，个人推荐直接嵌入`html`代码。

<div id="custom_chart_1" class="highcharts"></div>
<style>
.highcharts-figure {
    min-width: 360px;
    max-width: 800px;
    margin: 0 auto;
}
#custom_chart_1 {
    height: 500px;
}
</style>

<script src="https://code.highcharts.com/maps/highmaps.js"></script>
<script src="https://code.highcharts.com/maps/modules/data.js"></script>
<script>
const data = [
    {
        name: 'United States of America',
        value: 1477
    },
    {
        name: 'Brazil',
        value: 490
    },
    {
        name: 'Mexico',
        value: 882
    },
    {
        name: 'Canada',
        value: 161
    },
    {
        name: 'Russia',
        value: 74
    },
    {
        name: 'Argentina',
        value: 416
    },
    {
        name: 'Bolivia',
        value: 789
    },
    {
        name: 'Colombia',
        value: 805
    },
    {
        name: 'Paraguay',
        value: 2011
    },
    {
        name: 'Indonesia',
        value: 372
    },
    {
        name: 'South Africa',
        value: 466
    },
    {
        name: 'Papua New Guinea',
        value: 1239
    },
    {
        name: 'Germany',
        value: 1546
    },
    {
        name: 'China',
        value: 54
    },
    {
        name: 'Chile',
        value: 647
    },
    {
        name: 'Australia',
        value: 62
    },
    {
        name: 'France',
        value: 844
    },
    {
        name: 'United Kingdom',
        value: 1901
    },
    {
        name: 'Venezuela',
        value: 503
    },
    {
        name: 'Ecuador',
        value: 1560
    },
    {
        name: 'India',
        value: 116
    },
    {
        name: 'Iran',
        value: 208
    },
    {
        name: 'Guatemala',
        value: 2716
    },
    {
        name: 'Philippines',
        value: 828
    },
    {
        name: 'Sweden',
        value: 563
    },
    {
        name: 'Saudi Arabia',
        value: 100
    },
    {
        name: 'Democratic Republic of the Congo',
        value: 87
    },
    {
        name: 'Kenya',
        value: 346
    },
    {
        name: 'Zimbabwe',
        value: 507
    },
    {
        name: 'Peru',
        value: 149
    },
    {
        name: 'Ukraine',
        value: 323
    },
    {
        name: 'Angola',
        value: 141
    },
    {
        name: 'Japan',
        value: 480
    },
    {
        name: 'United Republic of Tanzania',
        value: 187
    },
    {
        name: 'Costa Rica',
        value: 3153
    },
    {
        name: 'Algeria',
        value: 66
    },
    {
        name: 'Pakistan',
        value: 196
    },
    {
        name: 'Spain',
        value: 301
    },
    {
        name: 'Finland',
        value: 487
    },
    {
        name: 'Nicaragua',
        value: 1225
    },
    {
        name: 'Libya',
        value: 83
    },
    {
        name: 'Cuba',
        value: 1211
    },
    {
        name: 'Uruguay',
        value: 760
    },
    {
        name: 'Oman',
        value: 426
    },
    {
        name: 'Italy',
        value: 439
    },
    {
        name: 'Czech Republic',
        value: 1657
    },
    {
        name: 'Poland',
        value: 414
    },
    {
        name: 'New Zealand',
        value: 465
    },
    {
        name: 'Guyana',
        value: 594
    },
    {
        name: 'Panama',
        value: 1574
    },
    {
        name: 'Malaysia',
        value: 347
    },
    {
        name: 'Namibia',
        value: 136
    },
    {
        name: 'South Korea',
        value: 1145
    },
    {
        name: 'Honduras',
        value: 921
    },
    {
        name: 'Iraq',
        value: 233
    },
    {
        name: 'Thailand',
        value: 198
    },
    {
        name: 'Mozambique',
        value: 125
    },
    {
        name: 'Turkey',
        value: 127
    },
    {
        name: 'Iceland',
        value: 958
    },
    {
        name: 'Kazakhstan',
        value: 36
    },
    {
        name: 'Norway',
        value: 312
    },
    {
        name: 'Syria',
        value: 484
    },
    {
        name: 'Zambia',
        value: 118
    },
    {
        name: 'South Sudan',
        value: 132
    },
    {
        name: 'Egypt',
        value: 83
    },
    {
        name: 'Madagascar',
        value: 143
    },
    {
        name: 'North Korea',
        value: 681
    },
    {
        name: 'Denmark',
        value: 1885
    },
    {
        name: 'Greece',
        value: 589
    },
    {
        name: 'Botswana',
        value: 131
    },
    {
        name: 'Sudan',
        value: 43
    },
    {
        name: 'Croatia',
        value: 1233
    },
    {
        name: 'Bulgaria',
        value: 627
    },
    {
        name: 'El Salvador',
        value: 3282
    },
    {
        name: 'Belarus',
        value: 320
    },
    {
        name: 'Myanmar',
        value: 98
    },
    {
        name: 'Portugal',
        value: 700
    },
    {
        name: 'Switzerland',
        value: 1575
    },
    {
        name: 'The Bahamas',
        value: 6094
    },
    {
        name: 'Lithuania',
        value: 973
    },
    {
        name: 'Somalia',
        value: 97
    },
    {
        name: 'Chad',
        value: 47
    },
    {
        name: 'Ethiopia',
        value: 52
    },
    {
        name: 'Yemen',
        value: 108
    },
    {
        name: 'Morocco',
        value: 123
    },
    {
        name: 'Suriname',
        value: 353
    },
    {
        name: 'French Polynesia',
        value: 14110
    },
    {
        name: 'Nigeria',
        value: 59
    },
    {
        name: 'Uzbekistan',
        value: 125
    },
    {
        name: 'Afghanistan',
        value: 80
    },
    {
        name: 'Austria',
        value: 631
    },
    {
        name: 'Belize',
        value: 2061
    },
    {
        name: 'Israel',
        value: 2186
    },
    {
        name: 'Nepal',
        value: 328
    },
    {
        name: 'Uganda',
        value: 238
    },
    {
        name: 'Romania',
        value: 196
    },
    {
        name: 'Vietnam',
        value: 145
    },
    {
        name: 'Gabon',
        value: 171
    },
    {
        name: 'Mongolia',
        value: 28
    },
    {
        name: 'United Arab Emirates',
        value: 514
    },
    {
        name: 'Latvia',
        value: 675
    },
    {
        name: 'Belgium',
        value: 1354
    },
    {
        name: 'Hungary',
        value: 458
    },
    {
        name: 'Laos',
        value: 178
    },
    {
        name: 'Ireland',
        value: 581
    },
    {
        name: 'Central African Republic',
        value: 63
    },
    {
        name: 'Azerbaijan',
        value: 448
    },
    {
        name: 'Taiwan',
        value: 1147
    },
    {
        name: 'Dominican Republic',
        value: 745
    },
    {
        name: 'Solomon Islands',
        value: 1286
    },
    {
        name: 'Slovakia',
        value: 728
    },
    {
        name: 'Cameroon',
        value: 70
    },
    {
        name: 'Malawi',
        value: 340
    },
    {
        name: 'Vanuatu',
        value: 2543
    },
    {
        name: 'Mauritania',
        value: 29
    },
    {
        name: 'Niger',
        value: 24
    },
    {
        name: 'Liberia',
        value: 301
    },
    {
        name: 'Netherlands',
        value: 856
    },
    {
        name: 'Puerto Rico',
        value: 3237
    },
    {
        name: 'Tunisia',
        value: 187
    },
    {
        name: 'Fiji',
        value: 1532
    },
    {
        name: 'Jamaica',
        value: 2585
    },
    {
        name: 'Kyrgyzstan',
        value: 146
    },
    {
        name: 'Republic of the Congo',
        value: 79
    },
    {
        name: 'Ivory Coast',
        value: 85
    },
    {
        name: 'Republic of Serbia',
        value: 336
    },
    {
        name: 'Turkmenistan',
        value: 55
    },
    {
        name: 'Mali',
        value: 20
    },
    {
        name: 'New Caledonia',
        value: 1368
    },
    {
        name: 'Bosnia and Herzegovina',
        value: 469
    },
    {
        name: 'Lesotho',
        value: 791
    },
    {
        name: 'Tajikistan',
        value: 170
    },
    {
        name: 'Antarctica',
        value: 2
    },
    {
        name: 'Burkina Faso',
        value: 84
    },
    {
        name: 'Georgia',
        value: 316
    },
    {
        name: 'Senegal',
        value: 104
    },
    {
        name: 'Kiribati',
        value: 23428
    },
    {
        name: 'Sri Lanka',
        value: 294
    },
    {
        name: 'Bangladesh',
        value: 138
    },
    {
        name: 'Estonia',
        value: 425
    },
    {
        name: 'Jordan',
        value: 203
    },
    {
        name: 'Cambodia',
        value: 91
    },
    {
        name: 'Guinea',
        value: 65
    },
    {
        name: 'Slovenia',
        value: 794
    },
    {
        name: 'Northern Cyprus',
        value: 1623
    },
    {
        name: 'Greenland',
        value: 7
    },
    {
        name: 'Marshall Islands',
        value: 82873
    },
    {
        name: 'Swaziland',
        value: 814
    },
    {
        name: 'Haiti',
        value: 508
    },
    {
        name: 'Seychelles',
        value: 30769
    },
    {
        name: 'Djibouti',
        value: 561
    },
    {
        name: 'Eritrea',
        value: 129
    },
    {
        name: 'Armenia',
        value: 390
    },
    {
        name: 'Cook Islands',
        value: 46610
    },
    {
        name: 'Ghana',
        value: 44
    },
    {
        name: 'Macedonia',
        value: 393
    },
    {
        name: 'Cape Verde',
        value: 2232
    },
    {
        name: 'Maldives',
        value: 30201
    },
    {
        name: 'Singapore',
        value: 12690
    },
    {
        name: 'Guinea Bissau',
        value: 284
    },
    {
        name: 'Lebanon',
        value: 782
    },
    {
        name: 'Sierra Leone',
        value: 112
    },
    {
        name: 'Togo',
        value: 147
    },
    {
        name: 'Turks and Caicos Islands',
        value: 8439
    },
    {
        name: 'Burundi',
        value: 273
    },
    {
        name: 'Equatorial Guinea',
        value: 250
    },
    {
        name: 'Falkland Islands',
        value: 575
    },
    {
        name: 'Kuwait',
        value: 393
    },
    {
        name: 'Moldova',
        value: 213
    },
    {
        name: 'Rwanda',
        value: 284
    },
    {
        name: 'Benin',
        value: 54
    },
    {
        name: 'East Timor',
        value: 403
    },
    {
        name: 'Kosovo',
        value: 551
    },
    {
        name: 'Micronesia',
        value: 8547
    },
    {
        name: 'Qatar',
        value: 518
    },
    {
        name: 'Saint Vincent and the Grenadines',
        value: 15424
    },
    {
        name: 'Tonga',
        value: 8368
    },
    {
        name: 'Western Sahara',
        value: 23
    },
    {
        name: 'Guam',
        value: 9191
    },
    {
        name: 'Mauritius',
        value: 2463
    },
    {
        name: 'Montenegro',
        value: 372
    },
    {
        name: 'Northern Mariana Islands',
        value: 10776
    },
    {
        name: 'Albania',
        value: 146
    },
    {
        name: 'Bahrain',
        value: 5263
    },
    {
        name: 'British Virgin Islands',
        value: 26490
    },
    {
        name: 'Comoros',
        value: 1790
    },
    {
        name: 'French Southern and Antarctic Lands',
        value: 522
    },
    {
        name: 'Samoa',
        value: 1418
    },
    {
        name: 'Spratly Islands',
        value: 800000
    },
    {
        name: 'Svalbard',
        value: 64
    },
    {
        name: 'Trinidad and Tobago',
        value: 780
    },
    {
        name: 'American Samoa',
        value: 13393
    },
    {
        name: 'Antigua and Barbuda',
        value: 6778
    },
    {
        name: 'Cayman Islands',
        value: 11364
    },
    {
        name: 'Grenada',
        value: 8721
    },
    {
        name: 'Palau',
        value: 6536
    },
    {
        name: 'Palestinian Territories',
        value: 500
    },
    {
        name: 'Anguilla',
        value: 21978
    },
    {
        name: 'Bhutan',
        value: 52
    },
    {
        name: 'Dominica',
        value: 2663
    },
    {
        name: 'Guernsey',
        value: 25608
    },
    {
        name: 'Hong Kong',
        value: 1864
    },
    {
        name: 'Luxembourg',
        value: 773
    },
    {
        name: 'Saint Kitts and Nevis',
        value: 7663
    },
    {
        name: 'Saint Lucia',
        value: 3300
    },
    {
        name: 'Saint Pierre and Miquelon',
        value: 8264
    },
    {
        name: 'São Tomé and Príncipe',
        value: 2075
    },
    {
        name: 'Virgin Islands of the U.S.',
        value: 5780
    },
    {
        name: 'Wallis and Futuna',
        value: 14085
    },
    {
        name: 'Aruba',
        value: 5556
    },
    {
        name: 'Barbados',
        value: 2326
    },
    {
        name: 'Bermuda',
        value: 18657
    },
    {
        name: 'British Indian Ocean Territory',
        value: 16667
    },
    {
        name: 'Brunei',
        value: 190
    },
    {
        name: 'Faroe Islands',
        value: 718
    },
    {
        name: 'Gambia',
        value: 99
    },
    {
        name: 'Gibraltar',
        value: 153846
    },
    {
        name: 'Jan Mayen',
        value: 2653
    },
    {
        name: 'Jersey',
        value: 8621
    },
    {
        name: 'Macau',
        value: 35461
    },
    {
        name: 'Malta',
        value: 3165
    },
    {
        name: 'Isle of Man',
        value: 1748
    },
    {
        name: 'Montserrat',
        value: 9804
    },
    {
        name: 'Nauru',
        value: 47170
    },
    {
        name: 'Niue',
        value: 3846
    },
    {
        name: 'Paracel Islands',
        value: 129032
    },
    {
        name: 'Saint Barthelemy',
        value: 40000
    },
    {
        name: 'Saint Helena, Ascension and Tristan da Cunha',
        value: 2538
    },
    {
        name: 'Saint Martin',
        value: 18382
    },
    {
        name: 'Sint Maarten',
        value: 29412
    },
    {
        name: 'Tuvalu',
        value: 39063
    },
    {
        name: 'Wake Island',
        value: 153846
    }
];


const getGraticule = () => {
    const data = [];

    // Meridians
    for (let x = -180; x <= 180; x += 15) {
        data.push({
            geometry: {
                type: 'LineString',
                coordinates: x % 90 === 0 ? [
                    [x, -90],
                    [x, 0],
                    [x, 90]
                ] : [
                    [x, -80],
                    [x, 80]
                ]
            }
        });
    }
    
    // Latitudes
    for (let y = -90; y <= 90; y += 10) {
        const coordinates = [];
        for (let x = -180; x <= 180; x += 5) {
            coordinates.push([x, y]);
        }
        data.push({
            geometry: {
                type: 'LineString',
                coordinates
            },
            lineWidth: y === 0 ? 2 : undefined
        });
    }
    
    return data;
};

// Add flight route after initial animation
const afterAnimate = e => {
    const chart = e.target.chart;

    if (!chart.get('flight-route')) {
        chart.addSeries({
            type: 'mapline',
            name: 'Flight route, Amsterdam - Los Angeles',
            animation: false,
            id: 'flight-route',
            data: [{
                geometry: {
                    type: 'LineString',
                    coordinates: [
                        [4.90, 53.38], // Amsterdam
                        [-118.24, 34.05] // Los Angeles
                    ]
                },
                color: '#313f77'
            }],
            lineWidth: 2,
            accessibility: {
                exposeAsGroupOnly: true
            }
        }, false);
        chart.addSeries({
            type: 'mappoint',
            animation: false,
            data: [{
                name: 'Amsterdam',
                geometry: {
                    type: 'Point',
                    coordinates: [4.90, 53.38]
                }
            }, {
                name: 'LA',
                geometry: {
                    type: 'Point',
                    coordinates: [-118.24, 34.05]
                }
            }],
            color: '#313f77',
            accessibility: {
                enabled: false
            }
        }, false);
        chart.redraw(false);
    }
};


Highcharts.getJSON(
    'https://code.highcharts.com/mapdata/custom/world.topo.json',
    topology => {

        const chart = Highcharts.mapChart('custom_chart_1', {
            chart: {
                map: topology
            },
    
            title: {
                text: 'Airport density per country',
                floating: true,
                align: 'left',
                style: {
                    textOutline: '2px white'
                }
            },
    
            subtitle: {
                text: 'Source: <a href="http://www.citypopulation.de/en/world/bymap/airports/">citypopulation.de</a><br>' +
                    'Click and drag to rotate globe<br>',
                floating: true,
                y: 34,
                align: 'left'
            },
    
            legend: {
                enabled: false
            },
    
            mapNavigation: {
                enabled: true,
                enableDoubleClickZoomTo: true,
                buttonOptions: {
                    verticalAlign: 'bottom'
                }
            },
    
            mapView: {
                maxZoom: 30,
                projection: {
                    name: 'Orthographic',
                    rotation: [60, -30]
                }
            },
    
            colorAxis: {
                tickPixelInterval: 100,
                minColor: '#BFCFAD',
                maxColor: '#31784B',
                max: 1000
            },
    
            tooltip: {
                pointFormat: '{point.name}: {point.value}'
            },
    
            plotOptions: {
                series: {
                    animation: {
                        duration: 750
                    },
                    clip: false
                }
            },
    
            series: [{
                name: 'Graticule',
                id: 'graticule',
                type: 'mapline',
                data: getGraticule(),
                nullColor: 'rgba(0, 0, 0, 0.05)',
                accessibility: {
                    enabled: false
                },
                enableMouseTracking: false
            }, {
                data,
                joinBy: 'name',
                name: 'Airports per million km²',
                states: {
                    hover: {
                        color: '#a4edba',
                        borderColor: '#333333'
                    }
                },
                dataLabels: {
                    enabled: false,
                    format: '{point.name}'
                },
                events: {
                    afterAnimate
                },
                accessibility: {
                    exposeAsGroupOnly: true
                }
            }]
        });
    
        // Render a circle filled with a radial gradient behind the globe to
        // make it appear as the sea around the continents
        const renderSea = () => {
            let verb = 'animate';
            if (!chart.sea) {
                chart.sea = chart.renderer
                    .circle()
                    .attr({
                        fill: {
                            radialGradient: {
                                cx: 0.4,
                                cy: 0.4,
                                r: 1
                            },
                            stops: [
                                [0, 'white'],
                                [1, 'lightblue']
                            ]
                        },
                        zIndex: -1
                    })
                    .add(chart.get('graticule').group);
                verb = 'attr';
            }
    
            const bounds = chart.get('graticule').bounds,
                p1 = chart.mapView.projectedUnitsToPixels({
                    x: bounds.x1,
                    y: bounds.y1
                }),
                p2 = chart.mapView.projectedUnitsToPixels({
                    x: bounds.x2,
                    y: bounds.y2
                });
            chart.sea[verb]({
                cx: (p1.x + p2.x) / 2,
                cy: (p1.y + p2.y) / 2,
                r: Math.min(p2.x - p1.x, p1.y - p2.y) / 2
            });
        };
        renderSea();
        Highcharts.addEvent(chart, 'redraw', renderSea);
    
    }
);
</script>
