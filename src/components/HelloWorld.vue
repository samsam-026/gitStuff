<template>
<div class="hello">
    <div class="container">
        <div class="row">
            <div class="col-lg-12">
                <h4>{{msg}}</h4>
            </div>
        </div>
        <div class="row">
            <div class="col-lg-4"></div>
            <div class="col-lg-4">
                <form class="form-inline" v-on:submit.prevent="getDetails">
                    <input type="text" v-model="username" class="form-control mb-2 mr-sm-2"  placeholder="Username" autofocus>
                    <button type="button" v-on:click.prevent="getDetails" class="btn btn-primary mb-2" >Submit</button>
                </form>
            </div>
        </div>
        <div class="row" v-if="resultsFetching">
            <div class="col-lg-2"> </div>
            <div class="col-lg-8">
                <div class="loader"></div>
            </div>
        </div>
        <div class="row" v-if="results.length !== 0">
            <div class="col-lg-2"> </div>
            <div class="col-lg-8 text-justify">
                <div v-for="item of results" :key="item.sha">
                    <h3>{{item.repo}}<span style="font-size: 16px"> - {{item.date.toLocaleString()}}</span></h3>
                    <h5>{{item.message}}</h5>
                    <p><a v-bind:href="item.url" target="_blank">{{item.sha}}</a></p>
                    <hr />
                </div>
            </div>
        </div>
        <div class="row" v-if="resultsFetched && results.length === 0">
            <div class="col-lg-2"> </div>
            <div class="col-lg-8">
                <h3>This user has not made any commits</h3>
            </div>
        </div>
    </div>
</div>
</template>

<script>
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data() {
    return {
      username: "",
      results: [],
      resultsFetched: false,
      resultsFetching: false
    };
  },
  methods: {
    getDetails() {
      if (this.username !== "") {
        let url = "https://api.github.com/users/" + this.username + "/events";
        axios
          .get(url)
          .then(response => {
            this.resultsFetching = true;
            let allCommits = [];
            response.data.forEach(data => {
              if (data.type === "PushEvent") {
                data.payload.commits.forEach(commit => {
                  let item = {
                    date: new Date(data.created_at),
                    message: commit.message,
                    url: commit.url,
                    sha: commit.sha,
                    head: data.payload.head,
                    repo: data.repo.name
                  };
                  allCommits.push(item);
                });
              }
            });
            this.resultsFetching = false;
            this.resultsFetched = true;
            this.results = allCommits;
          })
          .catch(error => {
            if (error.toString() === "Error: Network Error") {
              alert("Please connect to the internet.");
            } else {
              alert("This user does not exist");
            }
          });
      } else {
        alert("Please type a username in the field.");
      }
    },
    renderChart() {
      var chart = this.$refs.chart;
      var ctx = chart.getContext("2d");
      let allData = {
        datasets: [
          {
            label: "Commits per repo",
            data: this.chartData,
            backgroundColor: [
              "rgba(255, 99, 132, 0.2)",
              "rgba(54, 162, 235, 0.2)",
              "rgba(255, 206, 86, 0.2)",
              "rgba(75, 192, 192, 0.2)",
              "rgba(153, 102, 255, 0.2)",
              "rgba(255, 159, 64, 0.2)"
            ],
            borderColor: [
              "rgba(255,99,132,1)",
              "rgba(54, 162, 235, 1)",
              "rgba(255, 206, 86, 1)",
              "rgba(75, 192, 192, 1)",
              "rgba(153, 102, 255, 1)",
              "rgba(255, 159, 64, 1)"
            ],
            borderWidth: 1
          }
        ],
        labels: this.chartLabels
      };
      new Chart(ctx, {
        type: "doughnut",
        data: allData,
        options: {
          responsive: false
        }
      });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->

<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

.loader {
  border: 16px solid #f3f3f3;
  border-radius: 50%;
  border-top: 16px solid #3498db;
  width: 120px;
  height: 120px;
  -webkit-animation: spin 2s linear infinite;
  /* Safari */
  animation: spin 2s linear infinite;
}

/* Safari */
@-webkit-keyframes spin {
  0% {
    -webkit-transform: rotate(0deg);
  }

  100% {
    -webkit-transform: rotate(360deg);
  }
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}
</style>
