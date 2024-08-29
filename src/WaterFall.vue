<script setup>
import { ref, onMounted, onUnmounted } from "vue";

const props = defineProps({
    gap: {
        type: Number,
        default: 20,
    },
    list: {
        type: Array,
    },
    itemClass: {
        type: String
    }
});

const container = ref();

let containerObserver;

let contentItems = []

const layout = []   // 已排列的每列高度

const calculate = (ele, width, height, index) => {
    const containerWidth = container.value.getBoundingClientRect().width
    const columns = Math.floor(containerWidth / width) 
    ele.style.width = width + 'px'
    ele.style.height = height + 'px'
    let positionIndex = 0
    if(index + 1 <= columns){
        positionIndex = index
    }else{
        positionIndex = layout.findIndex(val => val === Math.min(...layout))
    }
    ele.style.left = positionIndex % columns * width + (positionIndex % columns) * props.gap + 'px'
    if(index + 1 <= columns){
        ele.style.top = 0 + 'px'
    } else{
        ele.style.top = layout[positionIndex] + 'px'
    }
    layout[positionIndex] = (layout[positionIndex] || 0) + (height || ele.clientHeight || 0) + props.gap
}

onMounted(() => {
    containerObserver = new MutationObserver((mutationList) => {
        for (const mutation of mutationList) {
            
            if (mutation.type === 'childList' && mutation.addedNodes.length > 0 && mutation.addedNodes[0].classList && mutation.addedNodes[0].classList.contains(props.itemClass)) {
                const item = mutation.addedNodes[0]
                contentItems.push(item)
                const index = contentItems.length - 1
                item.style.position = 'absolute'
                item.style.backgroundColor = '#d8d8d8'
                item.style.transition = 'all 0.3s ease'
                calculate(mutation.addedNodes[0], item.clientWidth, item.clientWidth / props.list[index].width * props.list[index].height, index)
            }
        }
    })

    containerObserver.observe(container.value, {
        childList: true,
    })

    // window.addEventListener('resize', () => {
    //     contentItems.forEach((item, index) => {
    //         calculate(item, item.getBoundingClientRect().width, item.getBoundingClientRect().width / props.list[index].width * props.list[index].height, index)
    //     })
    // })
});

onUnmounted(() => {
    containerObserver.disconnect()
})
</script>

<template>
    <div class="waterfall-container" ref="container">
        <slot v-for="(item, index) in list" :key="index" :item="item" :index="index"></slot>
    </div>
</template>

<style scoped>
.waterfall-container {
    width: 100%;
    height: 100%;
    position: relative;
    overflow: scroll;
}
</style>
