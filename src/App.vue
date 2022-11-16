<template>
    <h1>Create new AppleDB entry</h1>

    <p>This site is a tool to quickly create database entries on <a href="https://appledb.dev" target="_blank">appledb.dev</a>.</p>

    <br>

    <div class="btnWrapper">
        <div class="btn" v-on:click="fetchDbInfo()">Request all database info</div>
        <div class="btn" v-on:click="showDeviceMaps = !showDeviceMaps">{{ showDeviceMaps ? 'Hide' : 'Expand'}} device maps</div>
    </div>

    <template v-if="appledb">
        <h2>Presets</h2>

        <div class="btnWrapper">
            <div class="btn" v-on:click="createPresets(betaPreset)">Create new betas</div>
            <div class="btn" v-on:click="createPresets(rcPreset)">Create new RCs</div>
            <div class="btn" v-on:click="createPresets(stablePreset)">Create new stables</div>
        </div>
    </template>

    <h2 v-if="entries">Entries</h2>
    
    <div v-for="(entry, index) in entries" :key="entry" style="
        border-bottom: 1px solid var(--c-border);
        padding-bottom: 1em;
        margin-bottom: 1em;
    ">
        <template v-for="field in Object.keys(entry)" :key="field">
            <div>
                <template v-if="field == 'osStr'">
                    {{ field }}: 
                    <select v-model="entry.osStr">
                        <option v-for="option in osStrArr" :key="option">
                            {{ option }}
                        </option>
                    </select>
                </template>
                <template v-else-if="field == 'deviceMap' && showDeviceMaps">
                    {{ field }}:
                    <ul style="line-height: 1.7em;">
                        <li v-if="this.appledb">Filter selection: <select v-model="deviceTypeFilter">
                            <option v-for="option in deviceTypeArr" :key="option">
                                {{ option }}
                            </option>
                        </select></li>
                        <li v-for="(device, index) in entry.deviceMap" :key="index">
                            <template v-if="appledb">
                                <select v-model="entry.deviceMap[index]">
                                    <option v-for="option in appledb.device.filter(x => deviceTypeFilter ? x.type == deviceTypeFilter : true)" :key="option.key" :value="option.key">
                                        {{ option.name }}
                                    </option>
                                </select>
                            </template>
                            <input v-else v-model="entry.deviceMap[index]"/> <a v-on:click="entry.deviceMap = entry.deviceMap.filter((x, i) => i != index)">X</a>
                        </li>
                        <li><a v-on:click="entry.deviceMap.push('')">Add device</a></li>
                    </ul>
                </template>
                <template v-else-if="['sources','fieldToAdd'].includes(field)"></template>
                <template v-else>
                    {{ field }}: <input v-model="entry[field]"/> <a v-if="![
                        'osStr',
                        'version',
                        'build',
                        'released',
                        'deviceMap'
                    ].includes(field)" v-on:click="delete entry[field]">X</a>
                </template>
            </div>
        </template>
        <p style="margin-top: 4px;">Add field: <input v-model="entry.fieldToAdd"/> <a v-on:click="entry.fieldToAdd.length ? entry[entry.fieldToAdd] = '' : false">+</a></p>
        <p style="margin-top: 1em;"><a v-on:click="entries = entries.filter((x, i) => index != i)">Remove entry</a></p>
    </div>

    <div class="btnWrapper">
        <div class="btn" v-on:click="this.entries.push({
            osStr: 'iOS',
            version: '',
            build: '',
            released: new Date().toISOString().slice(0,10),
            deviceMap: ['iPhone15,2'],
            fieldToAdd: ''
        })">Create new entry</div>
        <div class="btn" v-if="this.entries.length" v-on:click="downloadEntries()">
            Download entries
        </div>
    </div>

    <br>

    <p>Created by <a href="https://github.com/emiyl" target="_blank">emiyl</a>.</p>
</template>

<script>
import { version } from '@vue/runtime-core'
export default {
    data() {
        return {
            entries: [],

            appledb: false,
            osStrArr: [
                "Bluetooth Headset Firmware",
                "Durian Firmware",
                "audioOS",
                "watchOS",
                "bridgeOS",
                "iOS",
                "iPadOS",
                "macOS",
                "tvOS"
            ],
            deviceTypeArr: [],
            deviceTypeFilter: false,
            showDeviceMaps: false,
            
            betaPreset: [
                {
                    osStr: 'macOS',
                    version: '11',
                    type: 'RC'
                },
                {
                    osStr: 'macOS',
                    version: '12',
                    type: 'RC'
                },
                {
                    osStr: 'watchOS',
                    version: '9',
                    type: 'beta'
                },
                {
                    osStr: 'iOS',
                    version: '16',
                    type: 'beta'
                },
                {
                    osStr: 'macOS',
                    version: '13',
                    type: 'beta'
                },
                {
                    osStr: 'tvOS',
                    version: '16',
                    type: 'beta'
                },
                {
                    osStr: 'audioOS',
                    version: '16',
                    type: 'beta'
                }
            ],
            
            rcPreset: [
                {
                    osStr: 'macOS',
                    version: '11',
                    type: 'RC'
                },
                {
                    osStr: 'macOS',
                    version: '12',
                    type: 'RC'
                },
                {
                    osStr: 'watchOS',
                    version: '9',
                    type: 'RC'
                },
                {
                    osStr: 'iOS',
                    version: '16',
                    type: 'RC'
                },
                {
                    osStr: 'macOS',
                    version: '13',
                    type: 'RC'
                },
                {
                    osStr: 'tvOS',
                    version: '16',
                    type: 'RC'
                },
                {
                    osStr: 'audioOS',
                    version: '16',
                    type: 'RC'
                }
            ],
            
            stablePreset: [
                {
                    osStr: 'macOS',
                    version: '11',
                    type: 'stable'
                },
                {
                    osStr: 'macOS',
                    version: '12',
                    type: 'stable'
                },
                {
                    osStr: 'watchOS',
                    version: '9',
                    type: 'stable'
                },
                {
                    osStr: 'iOS',
                    version: '16',
                    type: 'stable'
                },
                {
                    osStr: 'macOS',
                    version: '13',
                    type: 'stable'
                },
                {
                    osStr: 'tvOS',
                    version: '16',
                    type: 'stable'
                },
                {
                    osStr: 'audioOS',
                    version: '16',
                    type: 'stable'
                }
            ]
        }
    },
    methods: {
        fetchDbInfo() {
            fetch("https://api.appledb.dev/main.json")
                .then(response => response.json())
                .then(data => this.appledb = data)
                .then(data => this.osStrArr = [...new Set(this.appledb.ios.map(x => x.osStr))])
                .then(data => this.deviceTypeArr = [...new Set(this.appledb.device.map(x => x.type))])
        },
        createPresets(preset) {
            this.entries.push(...preset.map(p => {
                const beta = ['beta','RC'].includes(p.type)
                const recentVersion = this.appledb.ios.filter(os =>
                    os.osStr == p.osStr &&
                    os.version.startsWith(p.version) &&
                    os.beta == beta
                ).sort((a,b) => {
                    const date = [a,b].map(x => new Date(x.released))
                    if (date[0] < date[1]) return 1
                    if (date[0] > date[1]) return -1
                    return 0
                })[0]
                if (!recentVersion) return null

                let retVersion = ''
                
                if (p.type == 'beta' || (p.type == 'RC' && recentVersion.version.includes('RC'))) {
                    let versionNum = recentVersion.version.split(' ')[0]
                    let iteration = recentVersion.version.split(' ')[2]
                    iteration++

                    retVersion = [versionNum, p.type, iteration].join(' ')
                } else if (p.type == 'RC') {
                    let versionNum = recentVersion.version.split(' ')[0]
                    retVersion = [versionNum, p.type].join(' ')
                } else {
                    let versionNumArr = recentVersion.version.split('.')
                    retVersion = versionNumArr.join('.')
                }

                return {
                    osStr: p.osStr,
                    version: retVersion,
                    build: '',
                    released: new Date().toISOString().slice(0,10),
                    deviceMap: recentVersion.deviceMap,
                    fieldToAdd: ''
                }
            }))
        },
        download(path, filename) {
            const anchor = document.createElement('a')
            anchor.href = path
            anchor.download = filename
            document.body.appendChild(anchor)
            anchor.click()
            document.body.removeChild(anchor)
        },
        downloadEntries() {
            const validEntries = this.entries.filter(x => x.osStr && x.build)
            
            validEntries.map(x => {
                const entry = {...x}
                delete entry.fieldToAdd
                const fileContent = JSON.stringify(entry, null, 2)
                const blob = new Blob([fileContent], { type: 'application/json' })
                const url = URL.createObjectURL(blob)
                this.download(url, (validEntries.filter(y => y.build == x.build).length > 1 ? x.osStr + '-' : '') + x.build + '.json')
                URL.revokeObjectURL(url)
            })
        }
    }
}
</script>
