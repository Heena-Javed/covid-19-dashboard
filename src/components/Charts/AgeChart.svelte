<script>
  import { onMount } from 'svelte';
  import Chart from 'chart.js';

  let ageChartEle;

  function getAgeGroup(age) {
    let ageGroup;
    switch (true) {
      case age >= 0 && age <= 10:
        ageGroup = '0-10';
        break;
      case age >= 11 && age <= 20:
        ageGroup = '11-20';
        break;
      case age >= 21 && age <= 30:
        ageGroup = '21-30';
        break;
      case age >= 31 && age <= 40:
        ageGroup = '31-40';
        break;
      case age >= 41 && age <= 50:
        ageGroup = '41-50';
        break;
      case age >= 51 && age <= 60:
        ageGroup = '51-60';
        break;
      case age >= 61 && age <= 70:
        ageGroup = '61-70';
        break;
      default:
        ageGroup = '70+';
        break;
    }
    return ageGroup;
  }

  function addAgeGroup(e) {
    e.ageGroup = getAgeGroup(parseInt(e.ageEstimate));
    return e;
  }

  function countAgeGroup(inital, current) {
    inital[current.status] = inital[current.status] || {};
    if (inital[current.status][current.ageGroup]) {
      inital[current.status][current.ageGroup]++;
    } else {
      inital[current.status][current.ageGroup] = 1;
    }
    return inital;
  }

  function createArrayData(obj) {
    return [
      obj['0-10'] || 0,
      obj['11-20'] || 0,
      obj['21-30'] || 0,
      obj['31-40'] || 0,
      obj['41-50'] || 0,
      obj['51-60'] || 0,
      obj['61-70'] || 0,
      obj['70+'] || 0
    ];
  }

  function getChartData(data) {
    const statusAgeObject = data.map(addAgeGroup).reduce(countAgeGroup, {});
    const recovered = createArrayData(statusAgeObject.Recovered);
    const deceased = createArrayData(statusAgeObject.Deceased);
    const hospitalized = createArrayData(statusAgeObject.Hospitalized);
    return {
      recovered,
      deceased,
      hospitalized
    };
  }

  function createChart(rawData) {
    const ctx = ageChartEle.getContext('2d');
    const data = getChartData(rawData);
    const chart = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['0-10', '11-20', '21-30', '31-40', '41-50', '51-60', '61-70', '70+'],
        datasets: [
          {
            label: 'Hospitalized',
            backgroundColor: 'rgb(255,235,59)',
            data: data.hospitalized
          },
          {
            label: 'Recovered',
            backgroundColor: 'rgb(76, 175, 80)',
            data: data.recovered
          },
          {
            label: 'Deceased',
            backgroundColor: 'rgb(244, 67, 54)',
            data: data.deceased
          }
        ]
      },
      options: {
        tooltips: {
          mode: 'index',
          intersect: false
        },
        responsive: true,
        scales: {
          xAxes: [
            {
              stacked: true
            }
          ],
          yAxes: [
            {
              stacked: true
            }
          ]
        }
      }
    });
  }

  async function getData() {
    await fetch(`https://api.rootnet.in/covid19-in/unofficial/covid19india.org`)
      .then(r => r.json())
      .then(res => {
        if (res.success && res.data) {
          const rawPaitentDataWithAge = res.data.rawPatientData.filter(e => e.ageEstimate);
          createChart(rawPaitentDataWithAge);
        }
      });
  }
  onMount(() => {
    getData();
  });
</script>

<div class="chart-container">
  <h3 class="chart-title">
    Age Demography Chart
    <small>(we have partial age data)</small>
  </h3>
  <canvas bind:this="{ageChartEle}"></canvas>
</div>