<template>
  <div class="release-version">
    <div v-if="availableDocs.length > 1">
      <span>Version:</span>
      <md-select id="docs-select" v-model="currentDocs" @change="changeDocs">
        <md-option v-for="doc in availableDocs" :key="doc" :value="doc">{{ doc }}</md-option>
      </md-select>
    </div>
  </div>
</template>

<style lang="scss" scoped>
  .release-version {
    font-size: 15px;

    @media (max-width: 480px) {
      display: none;
    }

    > div {
      display: flex;
      align-items: center;
    }

    .md-select {
      width: auto;
      min-width: auto;
      margin: 0 8px;

      &:after {
        color: rgba(#fff, .87);
      }
    }
  }
</style>

<script>
  let versions = null;

  export default {
    data: () => ({
      latest: null,
      currentDocs: null,
      availableDocs: []
    }),
    methods: {
      changeDocs() {
        const location = window.location;
        const latestUrl = location.origin + '/' + location.hash;
        const releaseUrl = location.origin + '/releases/v' + this.currentDocs + '/' + location.hash;

        if (process.NODE_ENV === 'development') {
          return;
        }

        if (this.currentDocs === this.latest) {
          window.location.href = latestUrl;
        } else {
          window.location.href = releaseUrl;
        }
      },
      getVersions(callback) {
        if (!versions) {
          const request = new XMLHttpRequest();

          request.open('GET', '/versions.json', true);
          request.setRequestHeader('Content-Type', 'application/json');
          request.onload = function() {
            versions = JSON.parse(this.response);
            versions.reverse();
            callback(versions);
          };
          request.send();
        } else {
          callback(versions);
        }
      },
      setVersion(versions) {
        this.latest = versions[0];
        this.currentDocs = versions[0];
        this.availableDocs = versions;
      },
      setCurrentByLocation() {
        let normalizedPathname = location.pathname.replace(/\/|releases\/v/g, '');

        if (normalizedPathname && this.availableDocs.indexOf(normalizedPathname) >= 0) {
          this.currentDocs = normalizedPathname;
        }
      }
    },
    mounted() {
      this.getVersions((response) => {
        this.setVersion(response);
        this.setCurrentByLocation();
      });
    }
  };
</script>
