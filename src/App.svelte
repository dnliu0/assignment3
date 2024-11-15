<script>
  import * as d3 from 'd3';
	import {onMount} from 'svelte';
  
  import Map from './Map.svelte';
  import Histogram from './Histogram.svelte';
  import Heatmap from './Heatmap.svelte';
  import Scatterplot from './Scatterplot.svelte';
  import Histogram2 from './Histogram2.svelte';
  import BarChart from './BarChart.svelte';
    import Parallel from './Parallel.svelte';

	let data = [];
  let data2 = [];
  let fullData = [];
  let filter1 = [];
  let filter2 = [];
  let cuisine_list = ['American',
                      'Chinese',
                      'Pizza',
                      'Coffee/Tea',
                      'Mexican',
                      'Latin American',
                      'Bakery Products/Desserts',
                      'Japanese',
                      'Italian',
                      'Caribbean']
  // let var1 = "percentWhite";
  let var1 = "diversity";
  let var3 = 'max_cuisine';
  let var2 = 'count';
  let new_cuisine_list;
  let criteria = [];
  let uniqueAspects = ['Data Meet Purpose', 'Data Quality Check', 'Pay Access Decision'];
  let uniqueInfoUsed = ['Topic alignment',
                    'Comprehensive Coverage',
                    'Required Attribute',
                    'Data Ranges',
                    'Patterns',
                    'Granularity',
                    'Collection Timeframe',
                    'Update Frequency',
                    'Composition of Data Fields',
                    'Sample Data',
                    'Methods Used',
                    'Good Practices Adherence',
                    'Unusable Data',
                    'Completeness',
                    'Access to the Full Dataset',
                    'Suitable Format',
                    'High Quality Supplements',
                    'Data Provider Communication',
                    'Reputable Source',
                    'Recommendations',
                    'Popular Dataset',
                    'Methods Comply with Legal Requirements',
                    'Access Terms'];
  // ['Source reputation', 'User reviews', 'Metadata', 'Data collection conditions', 'Prevalence of missing data', 'Ability to access full datasets', 'Sample Data', 'Visualizations'];
  let userInput = [];
  let marks;
  let marks2;
  let question;
  let indicator;
  let brushLayer;
  let brush;

	onMount(async function() {    
    //load data from 
    let table = d3.csv('forassign3.csv', (d) => ({
          ...d,
          //'ZIPCODE': d['ZIPCODE'].toString().slice(0, 5),
          // 'Information Used' : d['Information Used'].split(', '), 
          // 'Information Used_N' : d['Information Used_N'].split(', '), 
        }));

    
    await Promise.all([table]).then((values) => {
      // console.log(values);
      let table = values[0];
      let aspects = [];
      criteria = Object.keys(table[0]);
      let criteriaMentioned = [];
      for (let i = 0; i < 68; i++) {
        data.push({});
        data[i]['idx'] = i;
        let count = 0;
        Object.keys(table[0]).forEach(o => {
          if (o) {
            data[i][o] = +table[i][o];
            if (table[i][o] === "1") {
              count += 1;
              criteriaMentioned.push(o)
            }
          }
          
        });
        data[i]['count'] = count;
        data[i]['mentioned'] = criteriaMentioned;
        criteriaMentioned = [];

      }
      console.log(data);
      //console.log(data);
      fullData = [...data];
    });
	});
 

  function updateData(){

    let subInfo = uniqueInfoUsed.slice(filter1[0], filter1[1]);

    if (filter1.length > 0 && filter2.length > 0) {
      data = fullData.filter((d) => {
        d['mentioned'].forEach((m) => {
          return (subInfo.includes(m));
        })
      });
    } else if (filter1.length > 0 && filter2.length == 0) {
      let result = [];
      let result2 = [];
      // data2 = fullData.filter((d) => {
      //     // Check if any elements of subInfo are mentioned
          
      //     let isMentioned = subInfo.some((m) => d.mentioned.includes(m));
      //     if (isMentioned) {
      //       let temp = { ...d };
      //         // Update the mentioned property to include only elements that match subInfo
      //         temp.mentioned = d.mentioned.filter(m => subInfo.includes(m));
      //         return temp;
      //     }
      //     return isMentioned;
      // });
      data = fullData.filter((d) => {
        let isMentioned = subInfo.some((m) => d.mentioned.includes(m));
        // if (isMentioned) {
        //   result.push(d);
        // }
        return isMentioned;
      });
    } else if (filter1.length == 0 && filter2.length > 0) {
      
      data = fullData.filter((d) => (d.properties[var2] >= filter2[0] && d.properties[var2] < filter2[1]));
    } else {
      data = [...fullData];
      // new_cuisine_list =  Array.from(new Set(data.map(d => d.properties["max_cuisine"])));
    }
  }
  function getRandomInt(max) {
    return Math.floor(Math.random() * max);
  }

  function getQuestion() {
    indicator = getRandomInt(2) + 2;
    if (indicator == 1) {
      question = "Your task is: Check the air quality in Chicago";
    } else if (indicator == 2) {
      question = "Your task is: Analyze customer preferences for new product features";
    } else if (indicator == 3) {
      question = "Your task is: Build visualizations for storytelling about elections";
    }
  }
  getQuestion()
  function addNewDataPoint() {
    // Create a new data point based on user input
    const newDataPoint = {
      idx: fullData.length, // Use the next index value
      mentioned: [...userInput],
      count: userInput.length,
      // Add any additional properties or default values as needed
    };

    // Add the new data point to fullData
    if (indicator == 1) {
      console.log('check1');
      fullData = [...fullData, newDataPoint];
    } else if (indicator == 2) {
      console.log('check2');
      const middleIndex = Math.floor(fullData.length / 2);
    fullData = [
      ...fullData.slice(0, middleIndex),
      newDataPoint,
      ...fullData.slice(middleIndex)
    ];
    } else if (indicator == 3) {
      console.log('check3');
      fullData = [newDataPoint, ...fullData];
    }
    fullData = fullData.map((item, index) => {
    item.idx = index;
    return item;
  });
    data = fullData;
    d3.select(brushLayer).call(brush.move, null);
    console.log('Added new data point:', newDataPoint);
    console.log('Updated fullData:', fullData);
    d3.select(marks).selectAll("circle")
            .data(fullData)
            // .style("fill", "goldenrod")
            .transition()
            .duration(350)
            .delay(100)
            .attr("r", d => d === newDataPoint ? 10 : 6)
            .style("fill", d => d === newDataPoint ? "blue" : "goldenrod")
            .transition()
            .duration(750)
            .delay(500)
            .attr("r", 6)
            .style("fill", "goldenrod")
            // .attr("r", 10)
    console.log("testing");

    
    // d3.select(marks).selectAll("circle")
    // .data(fullData).exit().remove();
    
  }
</script>

<main>
  <!-- <h1>NYC Restaurants</h1> -->
  <div class="flex-container row">
    <!-- <div class="map"><Heatmap data={data} fullData={fullData}/></div> -->
    <div><Scatterplot data={data} fullData={fullData} criteria={criteria} bind:marks={marks} bind:marks2={marks2} /></div>
    <!-- <div><Histogram2 data={data} fullData={fullData} criteria={criteria}/></div> -->
    <div><BarChart data={data} fullData={fullData} criteria={criteria} update={updateData} bind:filter={filter1} bind:brushLayer={brushLayer} bind:brush={brush}/></div>
    <!-- <div><Parallel data={data} fullData={fullData} criteria={criteria} update={updateData} bind:filter={filter1}/></div> -->
  </div>
  <div class="flex-container row">
    <p>{question}</p>
    <div class="selection">
      {#each uniqueInfoUsed as c, i}
          <label>
            <input type="checkbox" bind:group={userInput} value={c} />
            {c}
          </label>
          {#if (i+1) % 4 == 0}
          <br/>
          {/if}
      {/each}
    </div>
  </div>
  <button on:click={addNewDataPoint}>Add A New Data Point</button>
</main>

<style>
  .selection {
    text-align: left;
  }
  .flex-container {
    display: flex;
    justify-content: center;  
    height: 100%;
    padding: 10px;
    gap: 0px;
  }

  .flex-container > div{
    padding: 8px;
  }

  .flex-container .row {
    flex-direction: row;  
  }

  .flex-container .col {
    flex-direction: column;  
  }

  .map { 
    flex-grow:1;
  }
			
  .hist { 
    flex-grow:0;
    margin-top: 10px;
  }

  p {
    margin-bottom: 10px;
    font-size: 1.1em;
    text-align: center;
  }

  .hist-text {
    margin-bottom: 10px;
  }

  
</style>