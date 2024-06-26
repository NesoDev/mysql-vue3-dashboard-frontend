<template>
    <div class="dropdown" :class="{ 'disabled': isBlocked }">
        <div class="select" @click='selectClicked'>
            <span class="selected">{{ optionSelect }}</span>
            <div class="caret" :class="{ 'rotate': menuOpen }">
                <img src="https://res.cloudinary.com/dimcnbuqs/image/upload/v1708632464/Vector_15_pgdeaa.png" alt="">
            </div>
        </div>
        <ul class="menu" v-if="menuOpen" :style="{ height: heightMenu }">
            <div class="menu-container">
                <li v-for="option, i in options" :key="i" @click="optionChange(option), optionSelected(option)">
                    {{ option }}
                </li>
            </div>
        </ul>
    </div>
</template>

<script setup>
import { onMounted, ref, defineProps, defineEmits, watch, onBeforeMount } from 'vue';
import { useStore } from 'vuex';

const isBlocked = ref(false)
const store = useStore();
const props = defineProps({
    options: {
        type: Array,
        require: true
    },
    name: {
        type: String,
        require: true
    },
    identifier: {
        type: String,
        require: true
    }
})

const emits = defineEmits(["optionIsChange"])

const optionChange = (option) => {
    if (option != optionSelect.value) {
        emits("optionIsChange", option)
    }
}

const menuOpen = ref(false);
const optionSelect = ref(props.name);

const selectClicked = () => {
    if (!isBlocked.value) {
        menuOpen.value = !menuOpen.value;
    }
}

const optionSelected = (option) => {
    optionSelect.value = option;
    menuOpen.value = false;
}
const clearSelection = () => {
    optionSelect.value = props.name;
};

const heightMenu = ref('')

onBeforeMount(() => {
    heightMenu.value = props.options.length < 3 ? 'auto': '96px'
    console.log(`dropdwon-${props.identifier}-heigth: ${heightMenu.value}`)
})

onMounted(() => {
    document.addEventListener('click', (event) => {
        const dropdown = document.getElementById(`select-${props.identifier}-container`);
        if (dropdown && !dropdown.contains(event.target)) {
            menuOpen.value = false
        }
    });

    watch(() => store.state.cleaned, () => {
        console.log('se detectó un cambio en cleaned')
        if (Array.isArray(store.state.cleaned) && store.state.cleaned.includes(props.identifier)) {
            console.log(`dropdown-${props.identifier} limpiado`)
            clearSelection();
        }
    });

    watch(() => store.state.blocked, () => {
        if (Array.isArray(store.state.blocked) && (store.state.blocked.includes(props.identifier) || store.state.blocked.includes('all'))) {
            isBlocked.value = true
        } else {
            isBlocked.value = false
        }
    });

    watch(() => props.options, () => {
        heightMenu.value = props.options.length < 3 ? 'auto': '96px'
    })
});
</script>

<style scoped>
.disabled {
    opacity: 0.5;
    pointer-events: none;
}

.dropdown {
    position: relative;
    border: 2px solid #1877F2;
    border-radius: 8px;
    outline: none;

    display: flex;
    flex-direction: row;
    justify-content: center;
}

.select {
    display: flex;
    align-items: center;
    cursor: pointer;
    width: 100%;
    height: 100%;
}

.selected {
    font-family: "Baloo 2", sans-serif;
    font-size: 16px;
    color: #5B5B5B;
    white-space: nowrap;
    max-width: 75%;
    height: 100%;
    padding: 0 0 0 10px;
    display: flex;
    align-items: center;
    overflow: hidden;
    text-overflow: ellipsis;
}

.caret {
    width: auto;
    height: auto;
    position: absolute;
    right: 10px;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
}

.caret img {
    width: 16px;
    height: 16px;
}

.rotate {
    transform: rotate(180deg);
}

.menu {
    font-family: "Baloo 2", sans-serif;
    font-size: 16px;
    color: #5B5B5B;
    list-style: none;
    padding: 0.5em 0.5em;
    background: #fefeff;
    box-shadow: 0 10px 20px #383838;
    border: 2px solid #1877F2;
    border-radius: 12px;
    position: absolute;
    top: 2.4em;
    left: 50%;
    width: 89%;
    transform: translateX(-50%);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 2;
}

.menu-container {
    width: 100%;
    height: 100%;
    overflow-y: auto;
    border-radius: 5px
}

.menu-container::-webkit-scrollbar {
    width: 0px;
    height: 0px;
}

.menu-container li {
    width: 89%;
    padding: 0.2em 0.5em;
    border-radius: 0.5em;
    cursor: pointer;
}

.menu li:hover {
    background: #4876d2;
    color: #fff
}
</style>