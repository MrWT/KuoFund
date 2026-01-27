<script setup>
    import { ref, reactive, onMounted, watch } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const props = defineProps({
        title: String,
        account: String
    })

    onMounted(() => {
        console.log("Survey mounted.");
        init();
    });

    let appState = ref("");
    let members = reactive([]);
    let locations = reactive([ "土城郭家", "北門雙春" ]);
    let selMemCode = ref("");
    let selLocation = ref("");

    // 初始化 component
    function init(){
        console.log("Survey.init");
        console.log("Survey.props.title", props.title);
        console.log("Survey.props.account", props.account);

        fetchInitData();
    }
    // 取得初始資料
    function fetchInitData(){
        // 取得成員
        let fetchPromise_members = fetchData({
            api: "get_members",
        }, "KUO-FUNDS");
        Promise.all([fetchPromise_members]).then((values) => {
            console.log("fetchPromise_members.values=", values);

            members.splice(0, members.length);
            values[0].forEach((memObj, mem_i) => {
                members.push({ code_name: memObj["code_name"], name: memObj["name"] });
            });

            members.sort((x, y) => {
                if(x["code_name"] > y["code_name"]) return 1;
                if(x["code_name"] < y["code_name"]) return -1;
                return 0;
            });
        });
    }

    // 監聽
    watch(selMemCode, (newValue, oldValue) => {
        console.log("watch.selMemCode=" + selMemCode.value);

        appState.value = "123";
    });
    watch(selLocation, (newValue, oldValue) => {
        console.log("watch.selLocation=" + selLocation.value);

        appState.value = "456";
    });
</script>

<template>

    <div class="w-1/1 h-1/1 flex flex-col gap-3">
        <div class="w-1/1 flex flex-col place-items-center">
            <div class="w-2/3 text-center text-2xl underline">
                2026 中秋聚餐地點問卷調查
            </div>
        </div>

        <div class="w-1/1 flex flex-col place-items-center">
            <div class="w-1/3 flex flex-row items-center gap-3">
                <span class="flex-none">姓名:</span>
                <select class="flex-1 border border-1" v-model="selMemCode">
                    <option v-for="(memObj, mem_i) in members" :value="memObj.code_name">{{ memObj.name }}</option>
                </select>
            </div>
        </div>

        <div class="w-1/1 flex flex-col place-items-center">
            <div class="w-1/3 flex flex-row items-center gap-3">
                <span class="flex-none">偏好地點:</span>
                <select class="flex-1 border border-1" v-model="selLocation">
                    <option v-for="(loc, loc_i) in locations" :value="loc">{{ loc }}</option>
                </select>
            </div>
        </div>

        <div class="w-1/1 flex flex-col place-items-center">
            <div class="w-1/1 text-center">
                {{ appState }}
            </div>
        </div>
    </div>

</template>

<style scoped>

</style>
