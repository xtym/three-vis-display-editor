vabse
<template>
  <div class="containerOperation-container">
    <div
      v-show="loading"
      class="loading-mask"
    >
      <vis-icon
        class="ani"
        code="#iconxuanzhuan"
      />
      <span>正在加载...</span>
    </div>

    <div
      v-if="!loading && !floderChildren.length"
      class="empty-tips"
    >
      <span>这里暂时是空的哦</span>
    </div>

    <div
      v-for="(item, index) in floderChildren"
      v-else
      :key="index"
      v-tooltip.bottom="`${item.name}`"
      class="file-item-box"
      :class="{ active: false }"
      :draggable="!item.dir"
      @click="chouseFile(item)"
      @dragstart="dragstart($event, item)"
    >
      <template v-if="item.dir">
        <vis-icon
          :size="iconSize"
          code="#iconwenjianjia"
        />
      </template>
      <template v-else>
        <img :src="item.preview">
      </template>

      <span
        class="item-title"
        v-text="item.name"
      />
      <vis-icon
        v-tooltip.bottom="`删除`"
        class="item-delete"
        size="16px"
        code="#iconshanchu"
        @click.native.stop="remove(item)"
      />
    </div>

    <div
      class="file-item-placeholder"
      :style="{
        display: floderChildren.length % 2 !== 0 ? 'block' : 'none',
        flex: 1,
      }"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      iconSize: "35px",
      fileIcon: {
        mtl: "#iconmtl",
      },

      configTemplateCache: {}, // 配置缓存
      configRootCache: {}, // 根物体缓存

      timer: "",
      throttleTime: 1000 / 45,
      canMove: true,
    };
  },
  computed: {
    loading() {
      return this.$store.getters["componentLibrary/loading"];
    },
    currentFloder() {
      return this.$store.getters["componentLibrary/currentFloder"];
    },
    floderChildren() {
      return this.currentFloder.children;
    },
    currentScene() {
      return this.$store.getters["scene/currentScene"];
    },
  },
  methods: {
    chouseFile(item) {
      if (item.dir) {
        this.$store.commit("componentLibrary/currentFloder", item);
      }
    },

    dragstart(event, item) {
      event.preventDefault();
      this.$store.commit("component/draggedComponentItem", item);
      this.$store.commit("component/dragging", true);

      const image = new Image();
      image.src = event.target.src;
      image.style.position = "fixed";
      image.style.zIndex = 20;
      image.style.opacity = 0.6;
      document.body.appendChild(image);
      const dragMove = ($event) => {
        if (this.canMove) {
          this.canMove = false;
          this.timer = setTimeout(() => {
            image.style.top = `${$event.clientY + 10}px`;
            image.style.left = `${$event.clientX + 10}px`;
            this.canMove = true;
          }, this.throttleTime);
        }
      };
      const dragOver = ($event) => {
        document.body.removeChild(image);
        document.body.removeEventListener("mousemove", dragMove);
        document.body.removeEventListener("mouseup", dragOver);
      };
      document.body.addEventListener("mousemove", dragMove);
      document.body.addEventListener("mouseup", dragOver);
    },

    // 删除文件
    remove(item) {
      return this.$tool.devTips();
      if (item.dir) {
        this.$confirm(`是否删除此分类: ${item.name}?`, "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }).then(() => {
          this.axios
            .post("/component/removeClassify", {
              id: item.id,
            })
            .then((res) => {
              if (res.status === 200) {
                this.$message({
                  type: "success",
                  message: "删除成功!",
                });
                this.$store.commit("componentLibrary/removeChildren", item);
              } else {
                this.$message.error(res.message);
              }
            });
        });
      } else {
        this.$confirm(
          `是否删除此模型: ${item.name}, 删除此模型相关的编辑项目会受到影响，是否继续?`,
          "提示",
          {
            confirmButtonText: "确定",
            cancelButtonText: "取消",
            type: "warning",
          }
        ).then(() => {
          this.axios
            .post("/component/removeComponent", {
              id: item.id,
            })
            .then((res) => {
              if (res.status === 200) {
                this.$message({
                  type: "success",
                  message: "删除成功!",
                });
                this.$store.commit("componentLibrary/removeChildren", item);
              } else {
                this.$message.error(res.message);
              }
            });
        });
      }
    },
  },
};
</script>

<style lang="less" scoped>
.containerOperation-container {
  .boxSetting(100%);
  background: @darkerTheme-backgroundColor;
  margin-bottom: @box-margin;
  padding: @box-padding;
  .flexLayout(row, space-around, center);
  flex-wrap: wrap;
  overflow: auto;
  max-height: 65vh;
  min-height: 100px;

  > .file-item-box {
    .flexLayout(column, space-between, center);
    .boxSetting(150px, 110px);
    overflow: hidden;
    margin: 0 0 @box-margin 0;
    cursor: pointer;
    .transitionSetting({background: @themeDarkHover-color;});
    &:hover .item-delete {
      opacity: 1;
    }
    position: relative;
    > *:not(.item-title) {
      height: calc(100% - 12px - 10px) !important;
      .flexLayout(row, center, center);
    }
    .item-title {
      width: 100%;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      text-align: center;
      font-size: 12px;
    }

    .item-delete {
      .absolutePosition(0, 0, unset, unset);
      height: unset !important;
      color: @error-color;
      opacity: 0;
    }
  }
  .active {
    position: relative;
    &::after {
      .absolutePosition(0, 0, unset, unset);
      content: "\2611";
      font-size: @title-fontSize;
      color: @theme-color;
      .boxSetting(@title-fontSize, @title-fontSize);
      text-align: center;
      line-height: @title-fontSize;
    }
  }
  > .loading-mask {
    .boxSetting();
    .flexLayout(column, center, center);
    .absolutePosition();
    color: @placeHolderText-color;
    .ani {
      animation: rotate 800ms ease infinite;
    }
  }
  @keyframes rotate {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }

  > .empty-tips {
    .boxSetting(100%, 150px);
    .flexLayout(column, center, center);
  }
}
</style>
