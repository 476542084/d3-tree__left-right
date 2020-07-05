<template>
  <div id="body">
    <button @click="addAnyNode">添加</button>
    <svg id="chart" class="width-100-percent">
      <g class="container" />
    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3'
import cloneDeep from '@/assets/js/lodash'
export default {
  data() {
    return {
      translate: [],
      selectedNode: null,
      chart: null,
      treeData: {
        name: 'Eve',
        children: [
          {
            name: 'Seth',
            children: [
              {
                name: 'Enos'
              },
              {
                name: 'Noam',
                children: [
                  {
                    name: 'Enos'
                  },
                  {
                    name: 'Noam'
                  }
                ]
              }
            ]
          },
          {
            name: 'Seth',
            children: [
              {
                name: 'Enos'
              },
              {
                name: 'Noam',
                children: [
                  {
                    name: 'Enos'
                  },
                  {
                    name: 'Noam'
                  }
                ]
              }
            ]
          },
          {
            name: 'Seth',
            children: [
              {
                name: 'Enos'
              },
              {
                name: 'Noam',
                children: [
                  {
                    name: 'Enos'
                  },
                  {
                    name: 'Noam'
                  }
                ]
              }
            ]
          },
          {
            name: 'Seth',
            children: [
              {
                name: 'Enos'
              },
              {
                name: 'Noam',
                children: [
                  {
                    name: 'Enos'
                  },
                  {
                    name: 'Noam'
                  }
                ]
              }
            ]
          }
        ]
      },
      margin: { top: 20, right: 90, bottom: 20, left: 90 },
      width: 0,
      height: 0,
      rectW: 45,
      rectH: 15
    }
  },
  mounted() {},
  methods: {
    //添加节点
    addAnyNode() {
      //未选中
      const source = {
        name: 'test',
        children: [
          { name: 'child1', value: '1' },
          { name: 'child3', value: '2' },
          { name: 'child3', value: '3' }
        ]
      }
      if (!this.selectedNode) {
        console.log('not select')
        return
      }
      let node = this.selectedNode
      if (node.children) {
        //某一节点(已展开)
        console.log('某一节点(已展开)')
        node.children.push(source)
      } else if (node._children) {
        //某一节点(未展开)
        console.log('某一节点(未展开)')
      } else {
        //某一叶子节点
        console.log('某一叶子节点')
        node.children = []
        node.children.push(source)
        // this.selectedNode.children.push(this.source)
      }
      this.selectedNode = node
      console.log('node', node)
      this.$nextTick(() => {
        this.chart.update(node)
      })
    },
    //删除节点
    delAnyNode() {},
    //选中节点
    handleClick(d) {
      this.selectedNode = d
      //展开/收缩
      if (d.children) {
        this.$set(d, '_children', d.children)
        d.children = null
      } else {
        this.$set(d, 'children', d._children)
        d._children = null
      }
      console.log('d', d)
      this.$nextTick(() => {
        this.chart.update()
      })
    },
    //初始化map
    initMap() {
      let that = this

      //初始化map参数
      let chart = this.MindMap()
        .width(this.width)
        .height(this.height)
        .text(function(d) {
          return d.name || d.text
        })

      let update = function(data = this.treeData) {
        let svgContainer
        //zoom缩放事件回调
        let zoomed = () => {
          svgContainer
            .select('g')
            .attr(
              'transform',
              `translate(${d3.event.translate[0]} ,${d3.event.translate[1]}) scale(${d3.event.scale})`
            )
        }
        //创建zoom缩放
        let zoom = d3.behavior
          .zoom() //缩放配置，
          .scaleExtent([0.5, 2]) //缩放比例
          .on('zoom', zoomed)
          //zoom缩放居中
          .translate([that.width / 2 + that.margin.left, that.margin.top])

        //挂载zoom缩放
        svgContainer = d3
          .select('#chart')
          .call(zoom)
          .on('dblclick.zoom', null)

        //svg:g居中
        svgContainer
          .select('g')

          .attr(
            'transform',
            `translate(${that.width / 2 + that.margin.left} ,${
              that.margin.top
            }) `
          )

        //挂载tree
        d3.select('#chart')
          .datum(data)
          .call(chart)
      }
      //首次加载
      update(this.treeData)
    },
    //主要实现
    MindMap() {
      let that = this,
        duration = 500,
        identity = '_id',
        text = function(d) {
          return d.name
        },
        idx = 0,
        enterNode = function(node) {
          node
            .append('rect')
            .attr('width', that.rectW)
            .attr('height', that.rectH)
            .attr('stroke', 'black')
            .attr('stroke-width', 1)
            .style('fill', function(d) {
              return d._children ? 'lightsteelblue' : '#fff'
            })
          node
            .append('text')
            .attr('x', that.rectW / 2)
            .attr('y', that.rectH / 2)
            .attr('dy', '.35em')
            .attr('text-anchor', 'middle')
            // .attr('transform', 'rotate(45 ' + rectW / 2 + ' ' + rectH / 2 + ')')
            .text(function(d) {
              return d.name
            })
        },
        updateNode = function(node) {
          node
            .select('rect')
            .attr('width', that.rectW)
            .attr('height', that.rectH)
            .attr('stroke', 'black')
            .attr('stroke-width', 1)
            .style('fill', function(d) {
              return d._children ? 'lightsteelblue' : '#fff'
            })

          node
            .select('text')
            .style('fill-opacity', 1)
            .style('fill', '#404080')
        },
        exitNode = function(node) {
          node
            .select('rect')
            .attr('width', that.rectW)
            .attr('height', that.rectH)
            .attr('stroke', 'black')
            .attr('stroke-width', 1)
        }

      var connector = this.elbow

      var chart = function(selection) {
        selection.each(function(root) {
          var container = d3.select(this)
          var vis = container
          var graphRoot = vis.select('g')
          that.graphRoot = graphRoot
          if (!graphRoot[0][0]) {
            vis = vis.append('svg:g')
          } else {
            vis = graphRoot
          }
          // 设置第一个元素的初始位置
          root.x0 = that.height / 2
          root.y0 = 0

          var tree = d3.layout
            .tree()
            // .separation(function(a, b) {
            //   return a.parent == root && b.parent == root ? 0.05 : -20
            // })
            .size([that.height, that.width])

          //更新回调
          chart.update = function() {
            container
              .transition()
              .duration(duration)
              .call(chart)
          }

          // 确保拥有左右节点
          if (!(root.left || root.right)) {
            var i = 0,
              l = (root.children || []).length
            root.left = []
            root.right = []
            for (; i < l; i++) {
              if (i % 2) {
                root.left.push(root.children[i])
                root.children[i].position = 'left'
              } else {
                root.right.push(root.children[i])
                root.children[i].position = 'right'
              }
            }
          }

          // 进行左右排列，计算新的树布局
          var nodesLeft = tree
            .size([that.height, that.width / 2 - 20])
            .children(function(d) {
              return d.depth === 0 ? d.left : d.children
            })
            .nodes(root)
            .reverse()
          var nodesRight = tree
            .size([that.height, that.width / 2])
            .children(function(d) {
              return d.depth === 0 ? d.right : d.children
            })
            .nodes(root)
            .reverse()
          root.children = root.left.concat(root.right)
          var nodes = (window.nodes = (function(left, right) {
            var root = right[right.length - 1]
            left.pop()
            left.forEach(function(node) {
              node.y = -node.y
              node.parent = root
            })
            return left.concat(right)
          })(nodesLeft, nodesRight))

          // 更新节点
          var node = vis.selectAll('g.node').data(nodes, function(d) {
            return d[identity] || (d[identity] = ++idx)
          })
          // 在父节点之前的位置输入任何新节点
          var nodeEnter = node
            .enter()
            .append('g')
            .attr('class', 'node')
            .attr('transform', function(d) {
              return 'translate(' + root.y0 + ',' + root.x0 + ')'
            })
            .on('click', that.handleClick)
          enterNode(nodeEnter)

          // 将节点转移到新的位置
          var nodeUpdate = node
            .transition()
            .duration(duration)
            .attr('transform', function(d) {
              return 'translate(' + d.y + ',' + d.x + ')'
            })

          updateNode(nodeUpdate)

          // 将退出节点转移到父节点的新位置.
          var nodeExit = node
            .exit()
            .transition()
            .duration(duration)
            .attr('transform', function() {
              return 'translate(' + root.y + ',' + root.x + ')'
            })
            .remove()

          exitNode(nodeExit)

          // 更新链接
          var link = vis
            .selectAll('path.link')
            .data(tree.links(nodes), function(d) {
              return d.target[identity]
            })

          // 在父节点之前的位置输入任何新链接
          link
            .enter()
            .insert('path', 'g')
            .attr('class', 'link')
            .attr('d', function() {
              var o = {
                x: root.x0,
                y: root.y0
              }
              return connector({
                source: o,
                target: o
              })
            })
            .transition()
            .duration(duration)
            // .attr('d', connector)
            .attr('d', function(d) {
              var s = {
                y: d.source.y + that.rectW,
                x: d.source.x + that.rectH / 2
              }
              var t = {
                x: d.target.x + that.rectH / 2,
                y: d.target.y
              }
              return connector({
                source: s,
                target: t
              })
            })

          // 连接新位置的过渡效果.
          link
            .transition()
            .duration(duration)
            .attr('d', connector)

          // 将退出节点转移到父节点的新位置。
          link
            .exit()
            .transition()
            .duration(duration)
            .attr('d', function(d) {
              var o = {
                x: root.x,
                y: root.y
              }
              return connector({
                source: o,
                target: o
              })
            })
            .remove()

          // 把旧的位置藏起来，以备转换(保留)
          nodes.forEach(function(d) {
            d.x0 = d.x
            d.y0 = d.y
          })
        })
      }

      chart.width = function(_) {
        if (!arguments.length) return that.width
        that.width = _
        return chart
      }

      chart.height = function(_) {
        if (!arguments.length) return that.height
        that.height = _
        return chart
      }

      chart.duration = function(_) {
        if (!arguments.length) return duration
        duration = _
        return chart
      }

      chart.connector = function(_) {
        if (!arguments.length) return connector
        connector = _
        return chart
      }

      chart.identity = function(_) {
        if (!arguments.length) return identity
        identity = _
        return chart
      }

      chart.text = function(_) {
        if (!arguments.length) return text
        text = _
        return chart
      }

      chart.nodeEnter = function(_) {
        if (!arguments.length) return enterNode
        enterNode = _
        return chart
      }

      chart.nodeUpdate = function(_) {
        if (!arguments.length) return updateNode
        updateNode = _
        return chart
      }

      chart.nodeExit = function(_) {
        if (!arguments.length) return exitNode
        exitNode = _
        return chart
      }
      this.chart = chart
      return chart
    },
    //链接绘线
    elbow(d) {
      var source = d.source
      var target = d.target
      var hy = (target.y - source.y) / 2
      return (
        'M' +
        source.y +
        ',' +
        source.x +
        'H' +
        (source.y + hy) +
        'V' +
        (target.x + this.rectH/2) +
        'H' +
        target.y
      )
    },
    //获取大小
    boxSize() {
      var w = window,
        d = document,
        e = d.documentElement,
        g = d.getElementsByTagName('body')[0],
        x = w.innerWidth || e.clientWidth || g.clientWidth,
        y = w.innerHeight || e.clientHeight || g.clientHeight
      return { width: x, height: y }
    }
  },
  created() {
    this.$nextTick(() => {
      if (process.client) {
        this.width =
          this.boxSize()['width'] - this.margin.left - this.margin.right
        this.height =
          this.boxSize()['height'] - this.margin.top - this.margin.bottom
        this.initMap()
      }
    })
  }
}
</script>
<style>
.node circle {
  cursor: pointer;
  fill: #fff;
  stroke: steelblue;
  stroke-width: 1.5px;
}

.node text {
  font-size: 11px;
}

path.link {
  fill: none;
  stroke: #ccc;
  stroke-width: 1.5px;
}

.node.selected circle {
  fill: green;
}
</style>
<style scoped>
* {
  touch-action: pan-y;
}
.width-100-percent {
  width: 100%;
  height: calc(100vh - 74px);
}
.container {
  /* width: 100%; */
  transform: translate(300, 0);
}
</style>


