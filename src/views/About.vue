<template>
  <div class="container">
    <div class="side">
      <el-button type="primary" size="mini" @click="setMode('default')"
        >只读操作</el-button
      >
      <el-button type="primary" size="mini" @click="getData"
        >获取当前数据</el-button
      >
      <el-collapse v-model="activeNames">
        <el-collapse-item title="节点" name="node">
          <div class="content">
            <el-button
              icon="el-icon-picture-outline-round"
              circle
              @click="setMode('addNode')"
            ></el-button>
          </div>
        </el-collapse-item>
        <el-collapse-item title="边" name="edge">
          <div class="content">
            <el-button
              icon="el-icon-minus"
              circle
              @click="setMode('addEdge')"
            ></el-button>
          </div>
        </el-collapse-item>
      </el-collapse>
    </div>
    <div class="main" id="container"></div>
    <div class="handle">
      <div class="node-edit" v-show="mode === 'node'">
        节点名称：<el-input
          v-model="nodeData.label"
          @change="changeNodeName"
        ></el-input>
      </div>
      <div class="edge-edit" v-show="mode === 'edge'">
        边名称：<el-input
          v-model="edgeData.label"
          @change="changeEdgeName"
        ></el-input>
      </div>
    </div>
  </div>
</template>

<script>
import G6 from "@antv/g6";
let graph = null;
export default {
  name: "About",
  data() {
    return {
      activeNames: ["node", "edge"],
      addedCount: 0,
      nodeData: {
        label: "",
        id: "",
      },
      edgeData: {
        label: "",
        id: "",
      },
      mode: "default",
    };
  },
  mounted() {
    this.initG6();
  },
  methods: {
    setMode(value) {
      graph.setMode(value);
    },
    initG6() {
      const container = document.getElementById("container");
      const width = container.scrollWidth;
      const height = (container.scrollHeight || 500) - 30;
      graph = new G6.Graph({
        container: "container",
        width,
        height,
        // The sets of behavior modes
        modes: {
          // Defualt mode
          default: ["drag-node", "click-select"],
          // Adding node mode
          addNode: ["click-add-node", "click-select"],
          // Adding edge mode
          addEdge: ["click-add-edge", "click-select"],
        },
        defaultNode: {
          /* node type */
          type: "circle",
          /* node size */
          size: [60],
          /* style for the keyShape */
          // style: {
          //   fill: '#9EC9FF',
          //   stroke: '#5B8FF9',
          //   lineWidth: 3,
          // },
          labelCfg: {
            /* label's position, options: center, top, bottom, left, right */
            position: "bottom",
            /* label's offset to the keyShape, 4 by default */
            //   offset: 12,
            /* label's style */
            //   style: {
            //     fontSize: 20,
            //     fill: '#ccc',
            //     fontWeight: 500
            //   }
          },
          /* icon configuration */
          icon: {
            /* whether show the icon, false by default */
            show: true,
            /* icon's img address, string type */
            // img: 'https://gw.alipayobjects.com/zos/basement_prod/012bcf4f-423b-4922-8c24-32a89f8c41ce.svg',
            /* icon's size, 20 * 20 by default: */
            //   width: 40,
            //   height: 40
          },
        },
        // The node styles in different states
        nodeStateStyles: {
          // The node styles in selected state
          selected: {
            stroke: "steelblue",
            lineWidth: 2,
          },
        },
        defaultEdge: {
          type: "line",
          /* configure the bending radius and min distance to the end nodes */
          style: {
            radius: 10,
            offset: 30,
            endArrow: true,
            /* and other styles */
            // stroke: '#F6BD16',
          },
        },
      });
      graph.data({
        nodes: [],
        edges: [],
      });
      graph.render();

      this.addNode();
      this.addEdge();
      graph.on("node:click", this.onNodeClick);
      graph.on("edge:click", this.onEdgeClick);
    },
    onNodeClick(e) {
      this.mode = "node";
      const item = e.item.getModel();
      this.nodeData.label = item.label;
      this.nodeData.id = item.id;
    },
    onEdgeClick(e) {
      this.mode = "edge";
      const item = e.item.getModel();
      this.edgeData.label = item.label;
      this.edgeData.id = item.id;
    },
    /**
     * @description 注册点击添加节点
     */
    addNode() {
      let _this = this;
      G6.registerBehavior("click-add-node", {
        // Set the events and the corresponding responsing function for this behavior
        getEvents() {
          // The event is canvas:click, the responsing function is onClick
          return {
            "canvas:click": "onClick",
          };
        },
        // Click event
        onClick(ev) {
          const self = this;
          const graph = self.graph;
          // Add a new node
          graph.addItem("node", {
            x: ev.canvasX,
            y: ev.canvasY,
            id: `node-${_this.addedCount}`, // Generate the unique id
            label: `node-${_this.addedCount}`,
          });
          _this.addedCount++;
        },
      });
    },
    /**
     * @description 添加边
     */
    addEdge() {
      G6.registerBehavior("click-add-edge", {
        // Set the events and the corresponding responsing function for this behavior
        getEvents() {
          return {
            "node:click": "onClick", // The event is canvas:click, the responsing function is onClick
            mousemove: "onMousemove", // The event is mousemove, the responsing function is onMousemove
            "edge:click": "onEdgeClick", // The event is edge:click, the responsing function is onEdgeClick
          };
        },
        // The responsing function for node:click defined in getEvents
        onClick(ev) {
          const self = this;
          const node = ev.item;
          const graph = self.graph;
          // The position where the mouse clicks
          // const point = { x: ev.x, y: ev.y };
          const model = node.getModel();
          if (self.addingEdge && self.edge) {
            graph.updateItem(self.edge, {
              target: model.id,
            });

            self.edge = null;
            self.addingEdge = false;
          } else {
            // Add anew edge, the end node is the current node user clicks
            self.edge = graph.addItem("edge", {
              source: model.id,
              target: model.id,
            });
            self.addingEdge = true;
          }
        },
        // The responsing function for mousemove defined in getEvents
        onMousemove(ev) {
          const self = this;
          // The current position the mouse clicks
          const point = { x: ev.x, y: ev.y };
          if (self.addingEdge && self.edge) {
            // Update the end node to the current node the mouse clicks
            self.graph.updateItem(self.edge, {
              target: point,
            });
          }
        },
        // The responsing function for edge:click defined in getEvents
        onEdgeClick(ev) {
          const self = this;
          const currentEdge = ev.item;
          if (self.addingEdge && self.edge === currentEdge) {
            self.graph.removeItem(self.edge);
            self.edge = null;
            self.addingEdge = false;
          }
        },
      });
    },
    onClick(ev) {
      const self = this;
      const node = ev.item;
      const graph = self.graph;
      // The position where the mouse clicks
      // const point = { x: ev.x, y: ev.y };
      const model = node.getModel();
      if (self.addingEdge && self.edge) {
        graph.updateItem(self.edge, {
          target: model.id,
        });

        self.edge = null;
        self.addingEdge = false;
      } else {
        // Add anew edge, the end node is the current node user clicks
        self.edge = graph.addItem("edge", {
          source: model.id,
          target: model.id,
        });
        self.addingEdge = true;
      }
    },
    changeNodeName() {
      if (this.nodeData.id) {
        const item = graph.findById(this.nodeData.id);
        graph.updateItem(item, {
          label: this.nodeData.label,
        });
      }
    },
    changeEdgeName() {
      if (this.edgeData.id) {
        const item = graph.findById(this.edgeData.id);
        graph.updateItem(item, {
          label: this.edgeData.label,
        });
      }
    },
    getData() {
      console.log(graph.save());
    },
  },
};
</script>

<style scoped>
.container {
  display: flex;
  width: 100%;
  height: 100%;
}
.side {
  width: 200px;
}
.main {
  flex: 1;
  background: #409eff;
}
.content {
  display: flex;
  align-items: center;
  justify-content: center;
}
.handle {
  width: 200px;
}
</style>