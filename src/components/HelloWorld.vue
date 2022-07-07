<template>
    <div class="flow-box">
        <!-- 左侧的   供拖拽图形 -->
        <div class="graph-stencil" ref="flowStencil"></div>
        <!-- 背景  画板 -->
        <div class="graph-container" ref="flowContainer"></div>
        <!-- 双击某个框时出现的   修改名称弹框 -->
        <name-drawer ref="nameDrawer" @editCellName="editCellName"></name-drawer>
        <!-- 右上角    操作栏 -->
        <toolbar ref="toolbar" @handleClick="handleClick" class="flow-tool"></toolbar>
        <slot name="rightPane">
            <modal :show.sync="showAttrConfig" title="属性配置" @keyup.stop.native="">
                <div slot="content">
                    <el-form
                        :model="nodeFrmData"
                        label-width="80px"
                        label-position="left">
                        <el-form-item label="名称" prop="name">
                            <el-input v-model="nodeFrmData.name"></el-input>
                        </el-form-item>
                        <el-form-item label="id" prop="id">
                            <el-input v-model="nodeFrmData.id"></el-input>
                        </el-form-item>
                        <el-form-item label="data" prop="data">
                            <el-input v-model="nodeFrmData.data"></el-input>
                        </el-form-item>
                    </el-form>
                </div>
                <span slot="footer">
                  <el-button @click="close">取消</el-button>
                  <el-button type="primary" @click="confirm">确定</el-button>
                  <el-button type="primary" @click="addPort">增加节点</el-button>
                  <el-button type="primary" @click="viewPort">查看端口</el-button>
                  <el-button type="primary" @click="viewEdge">查看连线</el-button>
                  <el-button type="primary" @click="getAllJson">转为json</el-button>
                </span>
            </modal>
        </slot>
        <div class="graph-minimapcontainer" ref="flowminimapContainer"></div>
        <!--节点的表格和表单-->
        <div>
            <el-dialog :visible.sync="portTableDialogVisible" width="30%">
                <el-table width="30%"
                          :data="allPort"
                >
                    <el-table-column
                        prop="id"
                        label="id"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.id }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="group"
                        label="group"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.group }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="data"
                        label="data"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.data }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        fixed="right"
                        label="操作"
                        width="100">
                        <template scope="scope">
                            <el-button @click="portTableRowEdit(scope.row,scope.$index)" type="text" size="small">修改
                            </el-button>
                            <el-button @click="portTableRowDelete(scope.row,scope.$index)" type="text" size="small">删除
                            </el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-dialog>
            <el-dialog title="修改" :visible.sync="PortFormDialogVisible">
                <el-form :model="editPortObj">
                    <el-form-item label="id">
                        <el-input v-model="editPortObj.id" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="group">
                        <el-input v-model="editPortObj.group" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="data">
                        <el-input v-model="editPortObj.data" auto-complete="off"></el-input>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button @click="PortFormDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="portTableRowEditDo">确 定</el-button>
                </div>
            </el-dialog>
        </div>
        <!--边的表格和表单-->
        <div>
            <el-dialog :visible.sync="edgeTableDialogVisible" width="30%">
                <el-table width="30%"
                          :data="allEdge"
                >
                    <el-table-column
                        prop="sourcePort"
                        label="sourcePort"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.sourcePort }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="targetPort"
                        label="targetPort"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.targetPort }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="data"
                        label="data"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.data }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        fixed="right"
                        label="操作"
                        width="100">
                        <template scope="scope">
                            <el-button @click="edgeTableRowEdit(scope.row,scope.$index)" type="text" size="small">修改
                            </el-button>
                            <el-button @click="edgeTableRowDelete(scope.row,scope.$index)" type="text" size="small">删除
                            </el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-dialog>
            <el-dialog title="修改" :visible.sync="edgeFormDialogVisible">
                <el-form :model="editPortObj">
                    <el-form-item label="id">
                        <el-input v-model="editPortObj.id" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="group">
                        <el-input v-model="editPortObj.group" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="data">
                        <el-input v-model="editPortObj.data" auto-complete="off"></el-input>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button @click="edgeFormDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="edgeTableRowEditDo">确 定</el-button>
                </div>
            </el-dialog>
        </div>

        <!--单个节点port的表单-->
        <div>
            <el-dialog :visible.sync="singlePortFormVisible" width="30%">
                <el-table width="30%"
                          :data="allEdge"
                >
                    <el-table-column
                        prop="sourcePort"
                        label="sourcePort"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.sourcePort }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="targetPort"
                        label="targetPort"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.targetPort }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="data"
                        label="data"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.data }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        fixed="right"
                        label="操作"
                        width="100">
                        <template scope="scope">
                            <el-button @click="edgeTableRowEdit(scope.row,scope.$index)" type="text" size="small">修改
                            </el-button>
                            <el-button @click="edgeTableRowDelete(scope.row,scope.$index)" type="text" size="small">删除
                            </el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-dialog>
            <el-dialog title="修改" :visible.sync="edgeFormDialogVisible">
                <el-form :model="editPortObj">
                    <el-form-item label="id">
                        <el-input v-model="editPortObj.id" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="group">
                        <el-input v-model="editPortObj.group" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="data">
                        <el-input v-model="editPortObj.data" auto-complete="off"></el-input>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button @click="edgeFormDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="edgeTableRowEditDo">确 定</el-button>
                </div>
            </el-dialog>
        </div>

        <el-tooltip content="xxx" value popper-class="tooltip-widget">
            <span style="position: relative; left: -1000px; top: -1000px"/>
        </el-tooltip>
    </div>
</template>

<script>
import {Graph, Shape, Addon} from "@antv/x6";
import {GridLayout} from "@antv/layout";
import "@antv/x6-vue-shape";
import "./shape";
import nameDrawer from "./nameDrawer"
import toolbar from "./toolbar"
import modal from "./modal"
import {DataUri} from '@antv/x6'
// import tableData from './tableData'
import testData from "../../public/data.json"


export default {
    components: {nameDrawer, toolbar, modal},
    data() {
        return {
            graph: null, //画布图层
            stencil: null, //组件栏组件
            cellId: null,//保存修改节点Id
            visible: true,
            message: 0,
            showAttrConfig: false,
            nodeFrmData: {},
            allPort: [],
            allEdge: [],
            port: {},
            portTableDialogVisible: false,
            PortFormDialogVisible: false,
            editPortObj: {
                id: "",
                group: "",
                data: ""
            },
            edgeTableDialogVisible: false,
            edgeFormDialogVisible: false,
            editEdgeObj: {
                sourcePort: "",
                targetPort: "",
                data: ""
            },
            singlePortFormVisible: false,
        };
    },
    mounted() {
        this.initGraph();
    },
    methods: {
        portTableRowEdit(row, index) {
            //记录索引
            this.listIndex = index;
            //记录数据
            this.editEdgeObj = row;
            //显示弹窗
            this.PortFormDialogVisible = true;
        },
        edgeTableRowEdit(row, index) {
            //记录索引
            this.listIndex = index;
            //记录数据
            this.editPortObj = row;
            //显示弹窗
            this.edgeFormDialogVisible = true;
        },
        portTableRowEditDo() {
            let index = this.listIndex
            //根据索引，赋值到list制定的数
            this.allPort[index] = this.editPortObj;
            //关闭弹窗
            this.PortFormDialogVisible = false;
        },
        edgeTableRowEditDo() {
            let index = this.listIndex
            //根据索引，赋值到list制定的数
            this.allEdge[index] = this.editEdgeObj;
            //关闭弹窗
            this.edgeFormDialogVisible = false;
        },
        openPortReport() {
            this.portTableDialogVisible = true
        },
        openEdgeReport() {
            this.edgeTableDialogVisible = true
        },
        portTableRowDelete(row) {
            this.allPort.splice(row, 1)
        },
        edgeTableRowDelete(row) {
            this.allEdge.splice(row, 1)
        },
        initGraph() {
            // #region 初始化画布
            this.graph = new Graph({
                container: this.$refs.flowContainer,
                //网格
                grid: {
                    size: 10,
                    visible: this.visible,
                    type: "doubleMesh",
                    args: [
                        {
                            color: "#E7E8EA",
                            thickness: 1,
                        },
                        {
                            color: "#CBCED3",
                            thickness: 1,
                            factor: 5,
                        },
                    ],
                },
                //开启 panning 选项来支持拖拽平移
                panning: {
                    enabled: true,
                    eventTypes: ["leftMouseDown", "rightMouseDown", "mousewheel"],
                },
                //滚轮缩放
                mousewheel: {
                    enabled: true,
                    zoomAtMousePosition: true,
                    minScale: 0.5,
                    maxScale: 3,
                },
                //全局连接设置
                connecting: {
                    router: "manhattan",
                    connector: {
                        name: "rounded",
                        args: {
                            radius: 8,
                        },
                    },
                    anchor: "center",
                    connectionPoint: "anchor",
                    allowBlank: false,
                    snap: {
                        radius: 20,
                    },
                    createEdge() {
                        return new Shape.Edge({
                            attrs: {
                                line: {
                                    stroke: "#000",
                                    strokeWidth: 1,
                                    targetMarker: { //target箭头设置
                                        name: "block",
                                        width: 12,
                                        height: 8,
                                    },
                                },
                            },
                            zIndex: 0,
                        });
                    },
                    validateConnection({targetMagnet}) {
                        return !!targetMagnet;
                    },
                },
                highlighting: {
                    magnetAdsorbed: {
                        name: "stroke",
                        args: {
                            attrs: {
                                fill: "#D06269",
                                stroke: "#D06269",
                            },
                        },
                    },
                },
                resizing: true,
                rotating: true,
                snapline: true,
                keyboard: true, //开启监听键盘
                history: true,
                clipboard: true,
                selecting: true, //通过点击或者套索框选节点
                scroller: {
                    enabled: true,
                },
                minimap: {
                    height: 100,
                    width: 200,
                    enabled: true,
                    container: this.$refs.flowminimapContainer,
                    scalable: true
                },
                onPortRendered(args) {
                    // console.log(args)
                    // console.log(args.port.id)
                    const contentContainer = args.contentContainer
                    // //略过标签时显示
                    // const labelContainer = args.labelContainer
                    // labelContainer.addEventListener("mouseenter", (e) => {
                    //     console.log("aaa")
                    // });
                    // this.viewSinglePort()
                    //如果是e，this就是graph.如果是(),就是contentContainer，也就是circle这个元素
                    contentContainer.addEventListener("mouseenter", function (e){
                        const node = args.node
                        const port = args.port
                        console.log("portID",port.id)
                        // console.log(args.getPortByID(args.node.port.id))
                        let theport = node.getPort(port.id)
                        theport.data  = 200
                        console.log("port",port)
                        console.log("theport",theport)
                        // aatemp.singlePortFormVisible = true
                        // const tooltip = document.querySelector(".tooltip-widget");
                        // if (tooltip) {
                        //     tooltip.innerHTML = args.port.id + '<br>' + args.port.data;
                        //     setTimeout(() => {
                        //         tooltip.style.left = `${
                        //             e.clientX - tooltip.offsetWidth / 2 + 5
                        //         }px`;
                        //         tooltip.style.top = `${e.clientY}px`;
                        //     }, 20);
                        // }
                    });
                    contentContainer.addEventListener("mouseleave", () => {
                        setTimeout(() => {
                            const tooltip = document.querySelector(".tooltip-widget");
                            if (tooltip) {
                                tooltip.style.left = "-1000px";
                                tooltip.style.top = "-1000px";
                            }
                        }, 30);
                    });
                    // console.log(this)
                    // this.viewSinglePort()
                },
            });
            // console.log(this)
            this.initStencil(); //初始化组件菜单栏 左边的
            this.initKeyboard(); //工具栏
            this.initEvent(); //鼠标移到组件上 显示连接点
        },
        change() {
            this.graph.drawGrid(
                {
                    type: 'mesh',
                }
            )
            this.graph.hideGrid()
            if (this.visible) {
                this.graph.hideGrid()
                this.visible = !this.visible
            } else {
                this.graph.showGrid()
                this.visible = !this.visible
            }
        },
        getNode() {
            let graph = this.graph
            // console.log(graph.toJSON())
            const nodes = graph.getNodes()
            const result = nodes.map(node => {
                const {id} = node.id
                // console.log("node--",node)
                // console.log("getData--",node.getData())
                const nodeId = node.getData().id || id
                const nodeName = node.getData().name || id
                const nodePort = node.port.ports
                return {
                    nodeId,
                    nodeName,
                    nodePort
                }
            })
            return result
        },
        getEdge() {
            let graph = this.graph
            // 获取所有边
            const edges = graph.getEdges()
            const result = edges.map(edge => {
                const {id} = edge.id
                const edgeId = edge.getData().id || id
                const edgeName = edge.getData().name || id
                return {
                    id: edge.id,
                    source: edge.source.port,
                    target: edge.target.port,
                    edgeId,
                    edgeName
                }
            })
            return result
        },
        getJson() {
            let atoms = {nodes: this.getNode(), edges: this.getEdge()}
            console.log(atoms)
            console.log("table:", this.allPort)
            return atoms
        },
        /*
        * 提供了一个类似侧边栏的 UI 组件，并支持分组、折叠、搜索等能力
        * */
        initStencil() {
            const {graph} = this;
            //初始化拖拽图形组件菜单栏
            this.stencil = new Addon.Stencil({
                title: "组件",
                target: graph,
                stencilGraphWidth: 200,
                stencilGraphHeight: 200,
                collapsable: true,
                groups: [
                    {
                        title: '基础流程图',
                        name: 'group1'
                    },
                    {
                        title: '网络拓扑图',
                        name: 'group2',
                        graphHeight: 250,
                        layoutOptions: {
                            rowHeight: 70,
                        },
                    },
                ],
                layoutOptions: {
                    columns: 2,
                    columnWidth: 60,
                    rowHeight: 60,
                    marginY: 20,
                },
                getDropNode(node) {
                    const size = node.size();
                    return node.clone().size(size.width * 3, size.height * 3);
                },
            });
            const stencilContainer = this.$refs.flowStencil;
            if (stencilContainer) {
                stencilContainer.appendChild(this.stencil.container);
            }
            this.initShape();
        },

        initShape() {
            //this其实是数据，这里是令graph = this.Grape
            const {graph, stencil} = this;
            this.port = {
                groups: {
                    po: {
                        position: {
                            name: 'ellipseSpread',
                            args: {
                                start: 0,
                            },
                        },
                        label: {
                            position: 'top',
                        },
                        attrs: {
                            circle: {
                                r: 10,
                                magnet: true,
                                stroke: '#31d0c6',
                                strokeWidth: 2,
                                fill: '#fff',
                            },
                        },
                    },
                },
                items: [
                    {
                        id: 'port1',
                        group: 'po',
                        attrs: {
                            text: {text: 'i1'},
                        },
                        data:100,
                    },
                ]
                //     {
                //         id: 'port2',
                //         group: 'po',
                //         attrs: {
                //             text: {text: 'i2'},
                //         },
                //     },
                //     {
                //         id: 'port3',
                //         group: 'po',
                //         attrs: {
                //             text: {text: 'i3'},
                //         },
                //     },
                //     {
                //         id: 'port4',
                //         group: 'po',
                //         attrs: {
                //             text: {text: 'o1'},
                //         },
                //     },
                //     {
                //         id: 'port5',
                //         group: 'po',
                //         attrs: {
                //             text: {text: 'o2'},
                //         },
                //     },
                // ],
            }
            Graph.registerNode(
                'custom-image',
                {
                    inherit: 'ellipse',
                    width: 60,
                    height: 60,
                    markup: [
                        {
                            tagName: 'ellipse',
                            selector: 'body',
                        },
                        {
                            tagName: 'image',
                        },
                        {
                            tagName: 'text',
                            selector: 'label',
                        },
                    ],
                    attrs: {
                        body: {
                            stroke: '#5F95FF',
                            fill: '#5F95FF',
                        },
                        image: {
                            width: 60,
                            height: 60,
                            refX: 13,
                            refY: 16,
                        },
                        label: {
                            refX: 3,
                            refY: 2,
                            textAnchor: 'left',
                            textVerticalAnchor: 'top',
                            fontSize: 12,
                            fill: '#fff',
                        },
                    },
                    ports: {...this.port},
                },
                true,
            )
            const r1 = graph.createNode({
                // shape: 'custom-rect',
                shape: 'ellipse',
                // label: "test",
                width: 60,
                height: 20,
                markup: [
                    {
                        tagName: 'rect',
                        selector: 'body',
                    },
                    {
                        tagName: 'text',
                        selector: 'label',
                    },
                    {
                        tagName: 'g',
                        children: [
                            {
                                tagName: 'text',
                                selector: 'btnText',
                            },
                            {
                                tagName: 'rect',
                                selector: 'btn',
                            },
                        ],
                    },
                ],
                attrs: {
                    // btn: {
                    //     refX: '100%',
                    //     refX2: -28,
                    //     y: 4,
                    //     width: 24,
                    //     height: 18,
                    //     rx: 10,
                    //     ry: 10,
                    //     fill: 'rgba(255,255,0,0.01)',
                    //     stroke: 'red',
                    //     cursor: 'pointer',
                    //     event: 'cell:delete',
                    // },
                    // btnText: {
                    //     fontSize: 14,
                    //     fill: 'red',
                    //     text: 'x',
                    //     refX: '100%',
                    //     refX2: -19,
                    //     y: 17,
                    //     cursor: 'pointer',
                    //     pointerEvent: 'none',
                    // },
                    body: {
                        fill: '#ffffff',
                        stroke: '#333333',
                        strokeWidth: 2,
                        refWidth: '100%',
                        refHeight: '100%',
                    },
                    label: {
                        fontSize: 14,
                        fill: '#333333',
                        refX: '50%',
                        refY: '50%',
                        textAnchor: 'middle',
                        textVerticalAnchor: 'middle',
                    },
                },
                ports: [
                    {
                        id: 'port-1',
                        attrs: {
                            // event: 'port:click',
                            // magnet: true,
                            circle: {
                                r: 3,
                                magnet: true,
                                stroke: '#31d0c6',
                                strokeWidth: 2,
                                fill: '#fff',
                            },
                            btn: {
                                refX: '100%',
                                refX2: -28,
                                y: 4,
                                width: 24,
                                height: 18,
                                rx: 10,
                                ry: 10,
                                fill: 'rgba(255,255,0,0.01)',
                                stroke: 'red',
                                cursor: 'pointer',
                                event: 'cell:delete',
                            },
                            btnText: {
                                fontSize: 14,
                                fill: 'red',
                                text: 'x',
                                refX: '100%',
                                refX2: -19,
                                y: 17,
                                cursor: 'pointer',
                                pointerEvent: 'none',
                            },
                            body: {
                                fill: '#ffffff',
                                stroke: '#333333',
                                strokeWidth: 2,
                                refWidth: '100%',
                                refHeight: '100%',
                            },
                            label: {
                                fontSize: 14,
                                fill: '#333333',
                                refX: '50%',
                                refY: '50%',
                                textAnchor: 'middle',
                                textVerticalAnchor: 'middle',
                            },
                            // event: console.log(this),
                            text: {
                                text: this.cell,
                            }
                        }
                    }]
            });
            const r2 = graph.createNode({
                shape: "custom-rect",
                attrs: {
                    body: {
                        rx: 6,
                        ry: 6,
                    },
                },
            });
            const r3 = graph.createNode({
                shape: "custom-polygon",
                attrs: {
                    body: {
                        refPoints: "0,10 10,0 20,10 10,20",
                    },
                },
            });
            const r4 = graph.createNode({
                shape: "custom-circle",
            });
            const r5 = graph.createNode({
                shape: 'custom-image',
                label: 'Client',
                attrs: {
                    image: {
                        'xlink:href': '../路由器.svg',
                    },
                },
            })
            stencil.load([r1, r2, r3, r4], 'group1');
            stencil.load([r5], 'group2');
        },

        initKeyboard() {
            const {graph} = this;
            // copy cut paste
            graph.bindKey(["meta+c", "ctrl+c"], () => {
                const cells = graph.getSelectedCells();
                if (cells.length) {
                    graph.copy(cells);
                }
                return false;
            });
            graph.bindKey(["meta+x", "ctrl+x"], () => {
                const cells = graph.getSelectedCells();
                if (cells.length) {
                    graph.cut(cells);
                }
                return false;
            });
            graph.bindKey(["meta+v", "ctrl+v"], () => {
                if (!graph.isClipboardEmpty()) {
                    const cells = graph.paste({offset: 32});
                    graph.cleanSelection();
                    graph.select(cells);
                }
                return false;
            });

            //undo redo
            graph.bindKey(["meta+z", "ctrl+z"], () => {
                if (graph.history.canUndo()) {
                    graph.history.undo();
                }
                return false;
            });
            graph.bindKey(["meta+shift+z", "ctrl+shift+z"], () => {
                if (graph.history.canRedo()) {
                    graph.history.redo();
                }
                return false;
            });

            // select all
            graph.bindKey(["meta+a", "ctrl+a"], () => {
                const nodes = graph.getNodes();
                if (nodes) {
                    graph.select(nodes);
                }
            });

            //delete
            graph.bindKey("backspace", () => {
                const cells = graph.getSelectedCells();
                console.log(cells)
                if (cells.length) {
                    graph.removeCells(cells);
                }
            });

            // zoom
            graph.bindKey(["ctrl+1", "meta+1"], () => {
                const zoom = graph.zoom();
                if (zoom < 1.5) {
                    graph.zoom(0.1);
                }
            });
            graph.bindKey(["ctrl+2", "meta+2"], () => {
                const zoom = graph.zoom();
                if (zoom > 0.5) {
                    graph.zoom(-0.1);
                }
            });
        },

        initEvent() {
            const {graph} = this;
            const container = this.$refs.flowContainer;
            //鼠标移到组件上 显示连接点
            const showPorts = (ports, show) => {
                for (let i = 0, len = ports.length; i < len; i = i + 1) {
                    ports[i].style.visibility = show ? "visible" : "hidden";
                }
            };
            //鼠标进入节点
            graph.on("node:mouseenter", () => {
                const ports = container.querySelectorAll(".x6-port-body");
                showPorts(ports, true);
            });
            //鼠标离开节点
            graph.on("node:mouseleave", () => {
                const ports = container.querySelectorAll(".x6-port-body");
                showPorts(ports, false);
            });
            //鼠标双击节点
            // graph.on("node:dblclick", ({ e, x, y, node }) => {
            //     this.cellId = node.id //保存节点Id
            //     let cell = node.getAttrs()
            //     let query = { //表单回显当前节点数据
            //         type: 'node',
            //         name: cell.label ? cell.label.text : '',
            //         fontColor: cell.label ? cell.label.fill : '',
            //         backgroundColor: cell.body ? cell.body.fill : '',
            //         borderColor: cell.body ? cell.body.stroke : '',
            //     }
            //     this.$refs.nameDrawer.openDrawer(query)
            // });
            // graph.on("edge:dblclick", ({ e, x, y, edge }) => {
            //     this.cellId = edge.id //保存节点Id
            //     const { attrs, labels } = edge.store.data //提取节点里的数据信息
            //     let label = labels && labels.length > 0 ? labels[0].attrs.label : null
            //
            //     let query = { //表单回显当前节点数据
            //         type: 'edge',
            //         name: label ? label.text : '',
            //         linkColor: attrs.line ? attrs.line.stroke : '',
            //         linkFontColor: label ? label.fill : '',
            //     }
            //     this.$refs.nameDrawer.openDrawer(query)
            // });
            graph.on("node:click", e => {
                let cell = e.cell
                console.log("node")
                if (cell) {
                    this.showAttrConfig = true;
                    console.log(cell.data)
                    console.log(cell._isNode)
                    console.log(cell._isEdge)
                    this.nodeFrmData = Object.assign(cell.data || {}, {
                        isNode: cell._isNode,
                        isEdge: cell._isEdge
                    });
                } else {
                    this.showAttrConfig = false;
                }
            })
            graph.on("edge:click", e => {
                let cell = e.cell
                if (cell) {
                    this.showAttrConfig = true;
                    this.nodeFrmData = Object.assign(cell.data || {}, {
                        isNode: cell._isNode,
                        isEdge: cell._isEdge
                    });
                } else {
                    this.showAttrConfig = false;
                }
            })
            graph.on("port:click", e => {
                console.log("bbbbbbbbbb")
                let cell = e.cell
                console.log("aaaaaaa")
                console.log(cell)
            })
            graph.on("cell:delete", ({view, e}) => {
                console.log(view.cell.ports)
                console.log(view)
                e.stopPropagation()
                // view.cell.remove()
            })
            // graph.on("node:ports:added",e => {
            //     console.log("hhhhhhhh")
            //     console.log(e)
            // })
            // graph.on("node:change:ports",e => {
            //     console.log("hhhhhhhh")
            //     console.log(e.current)
            // })
        },

        editCellName(form) {
            //修改节点信息
            let node = this.graph.getCellById(this.cellId)
            if (node.isNode()) { //判断是否是属性节点
                node.setAttrs( //设置节点属性
                    {
                        label: {text: form.name, fill: form.fontColor}, //修改字体名称/颜色
                        body: {fill: form.backgroundColor, stroke: form.borderColor} //修改背景色/边框色
                    },
                )
            } else {
                node.setAttrs( //设置节点属性
                    //连接线颜色
                    {line: {stroke: form.linkColor},},
                )
                node.setLabels({
                    attrs: { //连接线字体/颜色
                        label: {text: form.name, fill: form.linkFontColor},
                    },
                })
            }
        },
        handleClick(name) {
            const {graph} = this
            switch (name) {
                case 'zoomIn':
                    graph.zoom(0.1)
                    break
                case 'zoomOut':
                    graph.zoom(-0.1)
                    break
                case 'undo':
                    graph.history.undo()
                    break
                case 'redo':
                    graph.history.redo()
                    break
                case 'delete':
                    graph.clearCells()
                    break
                case 'center':
                    graph.centerContent()
                    break
                case 'export':
                    graph.toPNG((dataUri) => {
                            // 下载
                            DataUri.downloadDataUri(dataUri, 'chart.png')
                        },
                        {
                            backgroundColor: "#ffffff",
                            padding: {
                                top: 20,
                                right: 30,
                                bottom: 40,
                                left: 50,
                            },
                        }
                    )
                    break
                case 'change':
                    this.change()
                    break
                case 'getJson':
                    this.getJson()
                    break
                case 'graphToJson':
                    this.graphToJson()
                    break
                default:
                    break
            }
        },
        confirm() {
            let graph = this.graph;
            let selectedCell = graph.getSelectedCells()[0];
            selectedCell.setData(this.nodeFrmData);
            let style = selectedCell.style;
            // if (selectedCell.shape === "html") {
            //     let html = selectedCell.style.html;
            //     let temDom = document.createElement("div");
            //     temDom.innerHTML = style.html;
            //     temDom.querySelector('[attr="name"]').innerText = this.nodeFrmData.name;
            //     // selectedCell.id
            //     style.html = temDom.innerHTML;
            //     selectedCell.setStyle(style);
            // } else {
            //     console.log(selectedCell.data)
            //     console.log(selectedCell.getData())
            //     // selectedCell.style.label = this.nodeFrmData.name;
            // }
            // graph.refresh(selectedCell);
            this.showAttrConfig = false;
        },
        addPort() {
            let graph = this.graph;
            let selectedCell = graph.getSelectedCells()[0];
            let allPorts = selectedCell.getPorts()
            let length = allPorts.length
            // console.log(allPorts.length)
            // console.log(selectedCell.attrs)
            // console.log(this.port)
            const addpp = () => {
                selectedCell.addPort({
                    id: 'port' + ++length,
                    //不能用this.popo
                    group: 'po',
                    attrs: {
                        text: {
                            text: "n" + length,
                        },
                        event: 'port:click',
                    },
                    data: 100,
                })
            }
            addpp()
            console.log(allPorts)
        },
        viewPort() {
            this.portTableDialogVisible = true
            let graph = this.graph;
            let selectedCell = graph.getSelectedCells()[0];
            let allPorts = selectedCell.getPorts()
            this.allPort = []
            for (let i = 0; i < allPorts.length; i++) {
                let temp = {
                    id: allPorts[i].id,
                    group: allPorts[i].group,
                    data: allPorts[i].data
                }
                this.allPort.push(temp)
            }
            console.log(allPorts.length)
            this.openPortReport()
        },
        viewEdge() {
            // console.log("aaaaa")
            this.edgeTableDialogVisible = true
            // let graph = this.graph;
            // let selectedCell = graph.getSelectedCells()[0];
            let allEdges = this.getEdge()
            this.allEdge = []
            for (let i = 0; i < allEdges.length; i++) {
                let temp = {
                    sourcePort: allEdges[i].source,
                    targetPort: allEdges[i].target,
                    data: allEdges[i].edgeName
                }
                this.allEdge.push(temp)
            }
            console.log(allEdges)
            this.openEdgeReport()
        },
        viewSinglePort() {
            console.log("bb")
            this.singlePortFormVisible = true
        },
        close() {
            this.showAttrConfig = false;
        },
        layout(data) {
            // preprocess
            const model = {
                nodes: [],
                edges: []
            };
            let tmp;
            if (data.cells) {
                tmp = data.cells;
            } else if (data.nodes || data.edges) {
                tmp = [].concat(data.nodes, data.edges);
            }
            if (tmp) {
                tmp.forEach((item) => {
                    if (item.shape !== "edge") {
                        model.nodes.push({
                            id: item.id,
                            size: [item.size.width, item.size.height],
                            label: item.attrs.text.text
                        });
                    } else {
                        let sourceId = item.source;
                        let targetId = item.target;
                        if (typeof item.source === "object") {
                            sourceId = item.source.cell;
                        }
                        if (typeof item.target === "object") {
                            targetId = item.target.cell;
                        }
                        model.edges.push({
                            source: sourceId,
                            target: targetId
                        });
                    }
                });
            }

            // layout
            const gridLayout = new GridLayout({
                type: "grid",
                begin: [10, 10],
                width: 480,
                height: 260,
                sortBy: "label",
                rows: 3,
                cols: 3,
                nodeSize: [100, 100],
            });
            const layoutData = gridLayout.layout(model);

            // postprocess
            if (layoutData && layoutData.nodes) {
                layoutData.nodes.forEach((item) => {
                    const d = tmp.find((d) => d.id === item.id);
                    if (!d.position) {
                        d.position = {};
                    }
                    d.position.x = item.x;
                    d.position.y = item.y;
                });
            }
            return data;
        },
        getAllJson(){
            let graph = this.graph
            let graphToJsonData = graph.toJSON()
            console.log(graphToJsonData)
        },
        graphToJson() {

            // console.log("graphtoJson",graphToJsonData)
            // let atemp = graph.parseJSON(graphToJsonData)
            // console.log("parseJson:",atemp)
            // let btemp = graph.fromJSON(graphToJsonData)
            // console.log("fromJSON:",btemp)

            console.log("testData", testData)
            let targetData = this.layout(testData)
            console.log("targetData", targetData)
            this.graph.fromJSON(targetData)
        },
        // getPortByID(id) {
        //     let graph = this.graph
        //     return graph.getPort('port1')
        // }
    },
};
</script>

<style lang="scss">
#gridtest {
    position: absolute;
    padding: 10px 20px;
    display: flex;
    top: 0;
    right: 300px;
    align-items: center;
    width: 1px;
    //text-align: center;
    text-align: left;
}

.flow-box {
    width: 100%;
    height: 100vh;
    display: flex;
    position: relative;

    .flow-tool {
        position: absolute;
        top: 0;
        right: 0;
    }
}

.graph-stencil {
    position: relative;
    width: 214px;
    height: 100%;
    border-right: 1px solid #dfe3e8;
}

.graph-container {
    flex: 1;
    height: 100%;
}

.graph-minimapcontainer {
    position: fixed;
    z-index: 999;
    bottom: 20px;
    right: 20px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.x6-widget-stencil {
    background-color: #fff;
}

.x6-widget-stencil-title {
    background-color: #fff;
}

.x6-widget-stencil-group-title {
    background-color: #fff !important;
}

.x6-widget-transform {
    margin: -1px 0 0 -1px;
    padding: 0px;
    border: 1px solid #239edd;
}

.x6-widget-transform > div {
    border: 1px solid #239edd;
}

.x6-widget-transform > div:hover {
    background-color: #3dafe4;
}

.x6-widget-transform-active-handle {
    background-color: #3dafe4;
}

.x6-widget-transform-resize {
    border-radius: 0;
}

.x6-widget-selection-inner {
    border: 1px solid #239edd;
}

.x6-widget-selection-box {
    opacity: 0;
}

</style>
