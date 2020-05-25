<template>
  <div id="app" class="py-5">
    <b-modal id="modal-readme" title="Readme" size="lg" hide-footer>
      <b-container fluid>
        <b-tabs content-class="mt-3">
          <b-tab title="HTML" active>
            <div class="row">
              <b-col cols="6">
                <h4>Configuration</h4>
                <ul>
                  <li
                    v-for="i in readme.options"
                    :key="i"
                  >{{ i }}</li>
                </ul>
              </b-col>
              <b-col cols="6">
                <h4>Libraries</h4>
                <b-table-simple>
                  <b-tbody>
                    <b-tr
                      v-for="i in readme.libraries"
                      :key="i.name"
                    >
                      <b-th><a :href="i.url" target="_blank">{{ i.name }}</a></b-th>
                      <b-td>{{ i.version }}</b-td>
                    </b-tr>
                  </b-tbody>
                </b-table-simple>
              </b-col>
            </div>
          </b-tab>
          <b-tab title="TXT">
            <a :href="readme.url" target="_blank">{{ build.name }}-readme.txt</a>
            <hr>
            <pre>{{ readme.txt }}</pre>
          </b-tab>
        </b-tabs>
      </b-container>
    </b-modal>
    <b-container>
      <h1>FFmpeg Builds</h1>
      <p class="lead">FFmpeg is the leading multimedia framework to decode,
      encode, transcode, mux, demux, stream, filter and play. All builds require
      at least Windows 7 or Mac OS X 10.10.</p>
      <hr>
      <b-row>
        <b-col lg="3" md="4">
          <b-dropdown
            block
            variant="outline-secondary"
            :text="option.version.selected"
            menu-class="w-100"
          >
            <b-dropdown-form>
              <b-input
                placeholder="Search builds"
                v-model="option.version.filter"
              ></b-input>
            </b-dropdown-form>
            <b-dropdown-divider></b-dropdown-divider>
            <b-dropdown-group header="Release">
              <b-dropdown-item
                v-for="i in filterBuilds(option.version.filter, option.version.options.release)"
                :key="i"
                @click="selectVersion(i)"
                :active="option.version.selected == i"
              >{{ i }}</b-dropdown-item>
            </b-dropdown-group>
            <b-dropdown-group header="Git">
              <b-dropdown-item
                v-for="i in filterBuilds(option.version.filter, option.version.options.git)"
                :key="i"
                @click="selectVersion(i)"
                :active="option.version.selected == i"
              >{{ i }}</b-dropdown-item>
            </b-dropdown-group>
          </b-dropdown>
        </b-col>
        <b-col>
          <h4>Version</h4>
          <p>Release builds are published approximately every six months, while Git
          builds are compiled multiple times a week. Git builds should be
          preferred unless a specific release version needs to be targeted.</p>
        </b-col>
      </b-row>
      <b-row>
        <b-col lg="3" md="4">
          <b-dropdown
            variant="outline-secondary"
            :text="option.architecture.selected.text"
            block
            menu-class="w-100"
          >
            <b-dropdown-item
              v-for="i in option.architecture.options"
              :key="i.value"
              @click="setBuildProp(option.architecture, i)"
              :active="option.architecture.selected == i"
            >{{ i.text }}</b-dropdown-item>
          </b-dropdown>
        </b-col>
        <b-col>
          <h4>Architecture</h4>
          <p>Builds are provided for Windows and macOS, with very few
          configuration differences between the two. 32-bit Windows builds are
          provided for historical purposes, but will be removed in the
          future.</p>
        </b-col>
      </b-row>
      <b-row>
        <b-col lg="3" md="4">
          <b-dropdown
            variant="outline-secondary"
            :text="option.linking.selected.text"
            block
            menu-class="w-100"
          >
            <b-dropdown-item
              v-for="i in option.linking.options"
              :key="i.value"
              @click="setBuildProp(option.linking, i)"
              :active="option.linking.selected == i"
            >{{ i.text }}</b-dropdown-item>
          </b-dropdown>
        </b-col>
        <b-col>
          <h4>Linking</h4>
          <p>Static builds include all components linked into the executable,
          while shared builds rely on libraries (.dll or .dylib) to operate. Dev
          builds provide header files, and will be used in development with a
          shared build. Most users should use the static build, while developers
          need to use both the shared and dev builds.</p>
        </b-col>
      </b-row>
      <b-row>
        <b-col lg="3" md="4">
          <b-dropdown
            block
            variant="outline-secondary"
            :text="option.license.selected.text"
            menu-class="w-100"
          >
            <b-dropdown-item
              v-for="i in option.license.options"
              :key="i.value"
              @click="setBuildProp(option.license, i)"
              :active="option.license.selected == i"
              :disabled="i.disabled"
            >{{ i.text }}</b-dropdown-item>
          </b-dropdown>
        </b-col>
        <b-col>
          <h4>License</h4>
          <p>All nightly builds are licensed as GPL 3.0, while GPL 3.0 and LGPL
          3.0 builds are provided for releases. GPL 3.0 builds are more complete
          than LGPL 3.0 builds as they can include x264 and x265.</p>
        </b-col>
      </b-row>
    </b-container>
    <div class="bg-light my-4 py-5">
      <b-container>
        <b-row align-v="center">
          <b-col lg="4" xl="3" md="5">
            <b-dropdown
              size="lg"
              block
              split
              :split-href="buildURL(option)"
              text="Download Build"
              variant="primary"
              menu-class="w-100"
            >
              <b-dropdown-item-button v-b-modal.modal-readme @click="updateReadme(option)">Readme</b-dropdown-item-button>
              <b-dropdown-item target="_blank" :href="gitwebURL(option.version.selected)">Source Code</b-dropdown-item>
              <b-dropdown-item target="_blank" :href="hashURL(option)">SHA-256</b-dropdown-item>
            </b-dropdown>
          </b-col>
          <b-col md="7" lg="4"><h6 class="mb-0">Name</h6><div class="text-muted">{{ build.name }}.zip</div></b-col>
          <b-col md="5" lg="2"><h6 class="mb-0">Size</h6><div class="text-muted">{{ build.size }}</div></b-col>
          <b-col md="7" lg="2"><h6 class="mb-0">Modified</h6><div class="text-muted">{{ build.modified }}</div></b-col>
        </b-row>
      </b-container>
    </div>
  </div>
</template>

<script>
const axios = require('axios');
const uap = require('ua-parser-js');
const moment = require('moment');

export default {
  name: 'App',
  data: function() {
    var d = {
      option: {
        version: {
          options: {
            release: [],
            git: []
          },
          selected: '',
          filter: ''
        },
        architecture: {
          options: [
            {
              text: 'Windows 64-bit',
              value: 'win64'
            },
            {
              text: 'Windows 32-bit',
              value: 'win32'
            },
            {
              text: 'macOS 64-bit',
              value: 'macos64'
            }
          ],
          selected: {}
        },
        linking: {
          options: [
            {
              text: 'Static',
              value: 'static'
            },
            {
              text: 'Shared',
              value: 'shared'
            },
            {
              text: 'Dev',
              value: 'dev'
            }
          ],
          selected: {}
        },
        license: {
          options: [
            {
              text: 'GPL 3.0',
              value: 'gpl'
            },
            {
              text: 'LGPL 3.0',
              value: 'lgpl',
              disabled: true,
            }
          ],
          selected: {}
        }
      },
      lgpl: [],
      build: {
        name: '',
        size: '',
        modified: '',
        hash: ''
      },
      cachedBuilds: {},
      readme: {}
    };

    // defaults

    var ua = uap(navigator.userAgent);

    if (ua.os.name == "Windows" && ua.cpu.architecture == "ia32") {
      d.option.architecture.selected = d.option.architecture.options[1];
    } else if (ua.os.name == "Mac OS") {
      d.option.architecture.selected = d.option.architecture.options[2];
    } else {
      d.option.architecture.selected = d.option.architecture.options[0];
    }

    d.option.linking.selected = d.option.linking.options[0];
    d.option.license.selected = d.option.license.options[0];

    return d;
  },
  methods: {
    filterBuilds: function(filter, builds) {
      let filtered = [];
      for (let i = 0; i < builds.length; i++) {
        let build = builds[i];
        if (build.includes(filter)) {
          filtered.push(build);
        }
        if (filtered.length == 4) {
          break;
        }
      }
      return filtered;
    },
    forceLicense: function(selectedVersion) {
      if (this.lgpl.includes(selectedVersion)) {
        this.option.license.options[1].disabled = false;
      } else {
        this.option.license.selected = this.option.license.options[0];
        this.option.license.options[1].disabled = true;
      }
    },
    selectVersion: function(selected) {
      this.option.version.selected = selected;
      this.forceLicense(selected);
      this.updateBuild();
    },
    buildName: function(option) {
      let v = option.version.selected;
      let a = option.architecture.selected.value;
      let l = option.linking.selected.value;

      let buildName = `ffmpeg-${v}-${a}-${l}`
      if (option.license.selected.value == 'lgpl') {
        buildName += '-lgpl';
      }

      return buildName;
    },
    updateBuild: function() {
      let buildName = this.buildName(this.option);

      let cached = this.cachedBuilds[buildName];
      if (cached) {
        this.build.name = cached.name;
        this.build.modified = cached.modified;
        this.build.size = cached.size;
        return;
      }

      let a = this.option.architecture.selected.value;
      let l = this.option.linking.selected.value;

      axios.head(`./${a}/${l}/${buildName}.zip`)
        .then(res => {
          let cn = res.headers['content-length']
          let mb = (cn / 1048576).toFixed(2); // 2^20
          if (mb > 1) {
            this.build.size = `${mb} MB`;
          } else {
            let kb = (cn / 1024).toFixed(2); // 2^10
            this.build.size = `${kb} KB`;
          }

          this.build.modified = moment(res.headers['last-modified']).format('lll');
          this.build.name = buildName;

          this.cachedBuilds[buildName] = {
            name: this.build.name,
            size: this.build.size,
            modified: this.build.modified
          };
        });
    },
    setBuildProp: function(prop, val) {
      prop.selected = val;
      this.updateBuild();
    },
    gitwebURL: function(ver) {
      if (ver.includes('-')) {
        return `https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/${ver.split('-')[1]}`;
      } else {
        return `https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/refs/tags/n${ver}`;
      }
    },
    updateReadme: function(option) {
      let buildName = this.buildName(option);

      let a = this.option.architecture.selected.value;
      let l = this.option.linking.selected.value;
      let url = `./readme/${a}/${l}/${buildName}-readme.txt`;

      axios.get(url)
        .then(res => {
          this.readme = this.parseReadme(res.data);
          this.readme.url = url
        })
    },
    parseReadme: function(readmeStr) {
      let readme = {
        options: [],
        libraries: [],
        txt: readmeStr
      }

      let lines = readmeStr.split('\r\n');

      for (let i = 0; i < lines.length; i++) {
        if (lines[i] == 'Configuration:') {
          for (let j = i + 1; lines[j] != ''; j++) {
            readme.options.push(lines[j].trim());
          }
          break;
        }
      }

      for (let i = 0; i < lines.length; i++) {
        if (lines[i] == 'Libraries:') {
          for (let j = i + 1; lines[j] != ''; j++) {
            let line = lines[j];
            let result = /\s{2}(.+?)\s{2,}(\S+)\s{2,}<(\S+)>/.exec(line);
            let lib = {
              name: result[1],
              version: result[2],
              url: result[3]
            };
            readme.libraries.push(lib);
          }
          break;
        }
      }

      return readme;
    },
    hashURL: function(option) {
      let buildName = this.buildName(option);

      let a = this.option.architecture.selected.value;
      let l = this.option.linking.selected.value;
      return `./sha256/${a}/${l}/${buildName}.zip.sha256`;
    },
    buildURL: function(option) {
      let buildName = this.buildName(option);

      let a = this.option.architecture.selected.value;
      let l = this.option.linking.selected.value;
      return `./${a}/${l}/${buildName}.zip`;
    }
  },
  mounted: function() {
    axios.get(`./builds.json?s=${Math.floor(Date.now() / 3600000)}`)
      .then(res => {
        this.option.version.options.release = res.data.release;
        this.option.version.options.git = res.data.git;
        this.option.version.selected = res.data.git[0];
        this.lgpl = res.data.lgpl;
        this.updateBuild();
      })
  }
}
</script>

<style>
</style>
