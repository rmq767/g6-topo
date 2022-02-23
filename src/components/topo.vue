<template>
  <div class="topo">
    <div id="topo"></div>
  </div>
</template>

<script>
import G6 from "@antv/g6";
export default {
  name: "topo-com",
  data() {
    return {
      topoData: {
        nodes: [
          {
            id: "node1",
            label: "node1",
            img: "cloud-province.png",
          },
          {
            id: "node2",
            label: "node2",
            img: "cloud-city.png",
          },
          {
            id: "node3",
            label: "node3",
            img: "cloud-county.png",
          },
        ],
        edges: [
          {
            source: "node1",
            target: "node2",
            label: "node1->node2",
            status: 1,
          },
          {
            source: "node2",
            target: "node3",
            label: "node2->node3",
            status: 0,
          },
        ],
      },
      graph: null,
      config: {
        node: {
          textColor: "#000",
          textError: "red",
          size: [20, 20],
          fontSize: 12,
        },
        edge: {
          stroke: "#000",
          strokeError: "red",
          lineWidth: 2,
          fontSize: 10,
        },
      },
    };
  },
  watch: {
    topoData: {
      handler(value) {
        this.graph.changeData(value);
      },
      deep: true,
    },
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      this.initGraph();
      this.renderNode();
      this.renderEdge();
      this.graph.data(this.topoData);
      this.graph.render();

      this.initDrag();
    },
    initGraph() {
      const container = document.getElementById("topo");
      const width = container.scrollWidth;
      const height = container.scrollHeight || 800;
      const minimap = this.initMinimap();

      this.graph = new G6.Graph({
        container: container,
        width,
        height,
        plugins: [minimap],
        defaultNode: {
          type: "machine",
          size: this.config.node.size,
          labelCfg: {
            style: {
              fill: this.config.node.textColor,
              fontSize: this.config.node.fontSize,
            },
          },
        },
        defaultEdge: {
          type: "line-growth",
          labelCfg: {
            refY: -10,
            style: {
              fontSize: this.config.edge.fontSize,
              fill: this.config.edge.stroke,
            },
          },
          style: {
            lineWidth: this.config.edge.lineWidth,
            stroke: this.config.edge.stroke,
            endArrow: {
              path: G6.Arrow.triangle(8, 10, 0),
              fill: this.config.edge.stroke,
            },
          },
        },
        layout: {
          type: "force",
          alpha: 1,
          preventOverlap: true,
          linkDistance: 300,
          nodeStrenth: -1000,
          onTick: () => {},
          onLayoutEnd: () => {},
        },
        modes: {
          default: [
            {
              type: "drag-canvas",
              enableOptimize: true, //拖拽隐藏keyshape外的元素
            },
            {
              type: "zoom-canvas",
              enableOptimize: true,
            },
            {
              type: "tooltip", //提示框
              formatText(model) {
                const text = `
                <div class="tooltips">
                  <p>${model.label}</p>
                </div>
                `;
                return text;
              },
            },
          ],
        },
      });
    },
    initMinimap() {
      return new G6.Minimap({
        size: [160, 160],
        className: "minimap",
        type: "delegate", //渲染成本：default>keyShape>delegate
        delegateStyle: {
          fill: this.config.edge.stroke,
          stroke: this.config.edge.stroke,
        },
      });
    },
    renderNode() {
      let _this = this;
      G6.registerNode("machine", {
        // 画节点
        draw(cfg, group) {
          const shape = group.addShape("image", {
            attrs: _this.nodeAttrsSetting("image", cfg),
            name: "img",
            draggable: true, //不设置将无法拖拽
          });
          group.addShape("text", {
            attrs: _this.nodeAttrsSetting("text", cfg),
            name: "title",
            draggable: true, //不设置将无法拖拽
          });
          return shape;
        },
        // 添加动画
        afterDraw(cfg, group) {
          const img = group.findAllByName("img");
          const text = group.findAllByName("title");
          if (cfg.status === 0 && img[0]) {
            _this.nodeAnimation(img[0], text[0]);
          }
        },
        update(cfg, node) {
          const group = node.getContainer();
          const shapeNodeImage = group.getChildren()[0];
          const shapeNodeText = group.getChildren()[1];
          if (cfg.status === 0) {
            if (!shapeNodeImage.cfg.animating) {
              _this.nodeAnimation(shapeNodeImage, shapeNodeText);
            }
          } else {
            shapeNodeImage.stopAnimate();
            shapeNodeText.stopAnimate();
          }

          // 更新写在最后
          shapeNodeImage.attr(_this.nodeAttrsSetting("image", cfg));
          shapeNodeText.attr(_this.nodeAttrsSetting("text", cfg));
        },
      });
    },

    renderEdge() {
      let _this = this;
      G6.registerEdge("line-growth", {
        // 画边
        draw(cfg, group) {
          const shape = group.addShape("path", {
            attrs: _this.edgeAttrsSetting("path", cfg),
            name: "path-shape",
          });
          if (cfg.label) {
            group.addShape("text", {
              attrs: _this.edgeAttrsSetting("text", cfg),
              name: "text-shape",
            });
          }
          return shape;
        },
        // 添加动画
        afterDraw(cfg, group) {
          const shape = group.getChildren()[0]; //path
          if (cfg.status === 0) {
            _this.edgeDashAnimation(shape);
          }
        },
        update(cfg, edge) {
          const group = edge.getContainer();
          const shapeEdgePath = group.getChildren()[0];
          const shapeEdgeText = group.getChildren()[1];
          if (cfg.status === 0) {
            if (!shapeEdgePath.cfg.animating) {
              _this.edgeDashAnimation(shapeEdgePath);
            }
          } else {
            shapeEdgePath.stopAnimate();
            shapeEdgePath.attr("lineDash", null);
          }

          // 更新写在最后
          shapeEdgePath.attr(_this.edgeAttrsSetting("path", cfg));
          shapeEdgeText.attr(_this.edgeAttrsSetting("text", cfg));
        },
      });
    },

    initDrag() {
      this.graph.on("node:dragstart", (e) => {
        this.graph.layout();
        refreshDragedNodePosition(e);
      });
      this.graph.on("node:drag", (e) => {
        const forceLayout = this.graph.get("layoutController").layoutMethods[0];
        forceLayout.execute();
        refreshDragedNodePosition(e);
      });
      this.graph.on("node:dragend", (e) => {
        e.item.get("model").fx = null;
        e.item.get("model").fy = null;
      });
      function refreshDragedNodePosition(e) {
        const model = e.item.get("model");
        model.fx = e.x;
        model.fy = e.y;
      }
    },

    nodeAttrsSetting(type, cfg) {
      let [imageWidth, imageHeight] = this.config.node.size;
      let attr = {};
      if (type === "image") {
        attr = {
          x: -(imageWidth / 2), //  /2中心点，不然会有其他问题，比如拖拽点问题
          y: -(imageHeight / 2),
          height: imageHeight,
          width: imageWidth,
          img: require(`../assets/${cfg.img}`),
        };
      } else {
        // 文本移到中心
        let labelWidth = G6.Util.getTextSize(
          cfg.label,
          this.config.node.fontSize
        )[0];
        let gap = labelWidth / 2;
        attr = {
          textBaseline: "top",
          y: 20,
          x: -gap,
          lineHeight: 20,
          text: cfg.label,
          fill: this.config.node.textColor,
        };
      }
      return attr;
    },
    edgeAttrsSetting(type, cfg) {
      console.log(1);
      const startPoint = cfg.startPoint;
      const endPoint = cfg.endPoint;
      const stroke = this.config.edge.stroke;
      let attr = {};
      if (type === "path") {
        attr = {
          stroke,
          path: [
            ["M", startPoint.x, startPoint.y],
            ["L", endPoint.x, endPoint.y],
          ],
          lineWidth: this.config.edge.lineWidth,
          endArrow: {
            path: G6.Arrow.triangle(8, 10, 0),
            fill: stroke,
            stroke,
            lineWidth: 0.01,
          },
        };
      } else {
        attr = {
          textBaseline: "middle",
          textAlign: "center",
          text: cfg.label,
          fill: stroke,
          fontSize: this.config.edge.fontSize,
          x: startPoint.x - (startPoint.x - endPoint.x) / 2,
          y: startPoint.y - (startPoint.y - endPoint.y) / 2 + 10,
        };
      }
      return attr;
    },
    nodeAnimation(img, text) {
      img.animate(
        (ratio) => {
          return {
            opacity: Math.abs(1 - ratio),
          };
        },
        {
          duration: 3000,
          repeat: true,
        }
      );
      text.animate(
        (ratio) => {
          return {
            fill: ratio > 0.5 ? "red" : "#000",
          };
        },
        {
          duration: 3000,
          repeat: true,
        }
      );
    },
    edgeDashAnimation(path) {
      let index = 0;
      path.animate(
        () => {
          index++;
          if (index > 9) {
            index = 0;
          }
          const res = {
            lineDash: [4, 4],
            lineDashOffset: -index,
          };
          return res;
        },
        {
          duration: 3000,
          repeat: true,
        }
      );
    },
  },
};
</script>

<style scoped></style>
