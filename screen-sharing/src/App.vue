<script setup lang="ts">
import Peer from "peerjs";
import { ref } from "vue";
import { useNative } from "./native/use-native";
const { setTitle, openFile, onUpdateCounter, counterValue } = useNative();

const title = ref<string>("");
const updateTitle = () => {
    setTitle(title.value);
};
const getFilePath = async () => {
    console.log(await openFile());
};
onUpdateCounter((value: number) => {
    console.log(value);
    counterValue(value);
});

const screenSharingPreviewVideoRef = ref<HTMLVideoElement>();
const cameraSharingPreviewVideoRef = ref<HTMLVideoElement>();

const startScreenSharingPreview = async () => {
    navigator.mediaDevices
        .getDisplayMedia({ video: true, audio: false })
        .then((stream) => {
            if (screenSharingPreviewVideoRef.value) {
                screenSharingPreviewVideoRef.value.srcObject = stream;
                screenSharingPreviewVideoRef.value.onloadedmetadata = () => {
                    screenSharingPreviewVideoRef.value?.play();
                };
            }
        })
        .catch((err) => {
            console.log(err);
        });
};
const endScreenSharingPreview = async () => {
    if (screenSharingPreviewVideoRef.value) {
        (screenSharingPreviewVideoRef.value.srcObject as MediaStream)
            .getTracks()
            .forEach((track) => track.stop());
        screenSharingPreviewVideoRef.value.srcObject = null;
    }
};

const startCameraSharingPreview = async () => {
    navigator.mediaDevices
        .getUserMedia({ video: true, audio: false })
        .then((stream) => {
            if (cameraSharingPreviewVideoRef.value) {
                cameraSharingPreviewVideoRef.value.srcObject = stream;
                cameraSharingPreviewVideoRef.value.onloadedmetadata = () => {
                    cameraSharingPreviewVideoRef.value?.play();
                };
            }
        })
        .catch((err) => {
            console.log(err);
        });
};
const endCameraSharingPreview = async () => {
    if (cameraSharingPreviewVideoRef.value) {
        (cameraSharingPreviewVideoRef.value.srcObject as MediaStream)
            .getTracks()
            .forEach((track) => track.stop());
        cameraSharingPreviewVideoRef.value.srcObject = null;
    }
};

const localId = ref<string>("");
const remoteId = ref<string>("");
const selfCameraVideoRef = ref<HTMLVideoElement>();
const remoteCameraVideoRef = ref<HTMLVideoElement>();
let peer: Peer;
let stream: MediaStream;
const startPeer = async () => {
    stream = await navigator.mediaDevices.getUserMedia({
        video: true,
        audio: false,
    });
    if (selfCameraVideoRef.value) {
        selfCameraVideoRef.value.srcObject = stream;
        selfCameraVideoRef.value.onloadedmetadata = () => {
            selfCameraVideoRef.value?.play();
        };
    }

    peer = new Peer({});
    peer.on("open", (id) => {
        localId.value = id;
    });
    peer.on("connection", (conn) => {});
    peer.on("call", async (call) => {
        call.answer(stream);
        call.on("stream", (remoteStream) => {
            if (remoteCameraVideoRef.value) {
                remoteCameraVideoRef.value.srcObject = remoteStream;
                remoteCameraVideoRef.value.onloadedmetadata = () => {
                    remoteCameraVideoRef.value?.play();
                };
            }
        });
    });
};
const connectRemote = async () => {
    const conn = peer.connect(remoteId.value);
    const call = peer.call(remoteId.value, stream);
};

const endConnect = async () => {
    if (selfCameraVideoRef.value) {
        (selfCameraVideoRef.value.srcObject as MediaStream)
            .getTracks()
            .forEach((track) => track.stop());
        selfCameraVideoRef.value.srcObject = null;
    }
};
</script>

<template>
    <input type="text" v-model="title" />
    <button type="button" @click="updateTitle">IPC更新标题</button>
    <div>
        <button type="button" @click="getFilePath">打开文件</button>
    </div>
    <div class="container">
        <h1>屏幕分享预览</h1>
        <div>
            <button type="button" @click="startScreenSharingPreview">
                启动
            </button>
        </div>
        <div>
            <button type="button" @click="endScreenSharingPreview">停止</button>
        </div>
        <div>
            <video
                ref="screenSharingPreviewVideoRef"
                width="640px"
                height="480px"
            ></video>
        </div>
        <h1>摄像头分享预览</h1>
        <div>
            <button type="button" @click="startCameraSharingPreview">
                启动
            </button>
        </div>
        <div>
            <button type="button" @click="endCameraSharingPreview">停止</button>
        </div>
        <div>
            <video
                ref="cameraSharingPreviewVideoRef"
                width="640px"
                height="480px"
            ></video>
        </div>
    </div>
    <div class="container">
        <div>
            <p>本地id: {{ localId }}</p>
            <input type="text" v-model="remoteId" placeholder="目标id" />
        </div>
        <h1>屏幕分享</h1>
        <h1>摄像头分享</h1>
        <div>
            <button type="button" @click="startPeer">启动</button>
            <button type="button" @click="connectRemote">连接</button>
            <button type="button" @click="endConnect">断开</button>
        </div>
        <div>
            <video
                ref="selfCameraVideoRef"
                width="640px"
                height="480px"
            ></video>
            <video
                ref="remoteCameraVideoRef"
                width="640px"
                height="480px"
            ></video>
        </div>
    </div>
</template>

<style scoped>
.container {
    border: dashed dodgerblue;
    width: 100%;
    padding: 2rem;
}
</style>
