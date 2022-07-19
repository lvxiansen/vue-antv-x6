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
            <modal :show.sync="nodeShowAttrConfig" title="节点属性配置" @keyup.stop.native="">
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
                        <el-form-item label="layer" prop="layer">
                            <el-input v-model="nodeFrmData.layer"></el-input>
                        </el-form-item>
                        <el-form-item label="province" prop="province">
                            <el-input v-model="nodeFrmData.province"></el-input>
                        </el-form-item>
                    </el-form>
                </div>
                <span slot="footer">
                  <el-button @click="nodeClose">取消</el-button>
                  <el-button type="primary" @click="confirmNode">确定</el-button>
                  <el-button type="primary" @click="addPort">增加节点</el-button>
                  <el-button type="primary" @click="viewPort">查看端口</el-button>
                  <el-button type="primary" @click="getAllJson">转为json</el-button>
                </span>
            </modal>
            <modal :show.sync="edgeShowAttrConfig" title="连线属性配置" @keyup.stop.native="">
                <div slot="content">
                    <el-form
                        :model="edgeFrmData"
                        label-width="80px"
                        label-position="left">
                        <el-form-item label="名称" prop="name">
                            <el-input v-model="edgeFrmData.name"></el-input>
                        </el-form-item>
                        <el-form-item label="id" prop="id">
                            <el-input v-model="edgeFrmData.id"></el-input>
                        </el-form-item>
                        <el-form-item label="bandwidth" prop="bandwidth">
                            <el-input v-model="edgeFrmData.bandwidth"></el-input>
                        </el-form-item>
                        <el-form-item label="delay" prop="delay">
                            <el-input v-model="edgeFrmData.delay"></el-input>
                        </el-form-item>
                    </el-form>
                </div>
                <span slot="footer">
                  <el-button @click="edgeClose">取消</el-button>
                  <el-button type="primary" @click="confirmEdge">确定</el-button>
<!--                  <el-button type="primary" @click="addPort">增加节点</el-button>-->
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
                        prop="layer"
                        label="layer"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.layer }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="province"
                        label="province"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.province }}</span>
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
            <el-dialog title="节点port修改" :visible.sync="PortFormDialogVisible">
                <el-form :model="editPortObj">
                    <el-form-item label="id">
                        <el-input v-model="editPortObj.id" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="group">
                        <el-input v-model="editPortObj.group" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="layer">
                        <el-input v-model="editPortObj.layer" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="province">
                        <el-input v-model="editPortObj.province" auto-complete="off"></el-input>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button @click="PortFormDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="portTableRowEditDo(editPortObj)">确 定</el-button>
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
                        prop="sourceNodeName"
                        label="sourceNodeName"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.sourceNodeName }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="targetNodeName"
                        label="targetNodeName"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.targetNodeName }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="sourcePortID"
                        label="sourcePortID"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.sourcePortID }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="targetPortID"
                        label="targetPortID"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.targetPortID }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="bandwidth"
                        label="bandwidth"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.bandwidth }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="delay"
                        label="delay"
                        width="100">
                        <template scope="scope">
                            <span>{{ scope.row.delay }}</span>
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
                        <el-input v-model="editPortObj.sourceName" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="group">
                        <el-input v-model="editPortObj.targetName" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="bandwidth">
                        <el-input v-model="editPortObj.bandwidth" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="delay">
                        <el-input v-model="editPortObj.delay" auto-complete="off"></el-input>
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
            <el-dialog title="单个端口属性" :visible.sync="singlePortFormVisible">
                <el-form :model="singlePort">
                    <el-form-item label="nodeID">
                        <el-input v-model="singlePort.nodeID" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="portID">
                        <el-input v-model="singlePort.portID" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="bandwidth">
                        <el-input v-model="singlePort.bandwidth" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="delay">
                        <el-input v-model="singlePort.delay" auto-complete="off"></el-input>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button @click="singlePortFormVisible = false">取 消</el-button>
                    <el-button type="primary" @click="singlePortEditDo(singlePort.nodeID,singlePort.portID)">确 定</el-button>
                </div>
            </el-dialog>
        </div>

        <el-tooltip content="xxx" value popper-class="tooltip-widget">
            <span style="position: relative; left: -1000px; top: -1000px"/>
        </el-tooltip>
    </div>
</template>

<script>
import {Graph, Shape, Addon,DataUri} from "@antv/x6";
import {GridLayout,ForceLayout} from "@antv/layout";
import "@antv/x6-vue-shape";
import "./shape";
import nameDrawer from "./nameDrawer"
import toolbar from "./toolbar"
import modal from "./modal"
// import tableData from './tableData'
import testData from "../../public/data.json"
import zhenshiData from "../../public/zhenshi.json"


export default {
    components: {nameDrawer, toolbar, modal,GridLayout},
    data() {
        return {
            graph: null, //画布图层
            stencil: null, //组件栏组件
            cellId: null,//保存修改节点Id
            visible: true,
            message: 0,
            nodeShowAttrConfig: false,
            edgeShowAttrConfig: false,
            nodeFrmData: {},
            edgeFrmData: {},
            allPort: [],
            allEdge: [],
            port: {},
            portTableDialogVisible: false,
            PortFormDialogVisible: false,
            editPortObj: {
                nodeID:"",
                id: "",
                group: "",
                data: ""
            },
            edgeTableDialogVisible: false,
            edgeFormDialogVisible: false,
            editEdgeObj: {
                sourceName: "",
                targetName: "",
                bandwidth:"",
                delay:"",
            },
            singlePortFormVisible: false,
            singlePort:{
                nodeID:"0",
                portID:"0",
                bandwidth:0,
                delay:0
            },
            publicPath: process.env.BASE_URL
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
            this.editPortObj = row;
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
            let graph = this.graph
            let index = this.listIndex
            //根据索引，赋值到list制定的数
            this.allPort[index] = this.editPortObj;
            const cell = graph.getSelectedCells()[0];
            cell.setPortProp(this.editPortObj.id,"data",this.editPortObj.data)
            //关闭弹窗
            this.PortFormDialogVisible = false;
        },
        edgeTableRowEditDo() {
            let graph = this.graph
            let index = this.listIndex
            //根据索引，赋值到list制定的数
            this.allEdge[index] = this.editEdgeObj;
            const cell = graph.getSelectedCells()[0];
            // cell.setData(this.editPortObj.bandwidth,"bandwidth",this.editPortObj.bandwidth)
            // cell.setData(this.editPortObj.delay,"delay",this.editPortObj.delay)
            cell.setData({bandwidth:this.editPortObj.bandwidth,delay:this.editPortObj.delay})
            //关闭弹窗
            this.edgeFormDialogVisible = false;
        },
        singlePortEditDo(nodeID,portID){
            const graph = this.graph
            const nodes = graph.getNodes();
            const anode = nodes.filter((node) => node.id === nodeID)[0];
            // let port = anode.getPort(portID)
            anode.setPortProp(portID,"bandwidth",this.singlePort.bandwidth)
            anode.setPortProp(portID,"delay",this.singlePort.delay)
            this.singlePortFormVisible = false;
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
            const vue = this
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
                    // this.viewSinglePort()
                    // 如果是e，this就是graph.如果是(),就是contentContainer，也就是circle这个元素
                    contentContainer.addEventListener("mouseenter", function (e){
                        const node = args.node
                        const port = args.port
                        console.log("portID",port.id)
                        // console.log(args.getPortByID(args.node.port.id))
                        let theport = node.getPort(port.id)
                        theport.data  = 200
                        // aatemp.singlePortFormVisible = true
                        const tooltip = document.querySelector(".tooltip-widget");
                        if (tooltip) {
                            tooltip.innerHTML = args.port.id + '<br>' + args.port.data;
                            setTimeout(() => {
                                tooltip.style.left = `${
                                    e.clientX - tooltip.offsetWidth / 2 + 5
                                }px`;
                                tooltip.style.top = `${e.clientY}px`;
                            }, 20);
                        }
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
                    contentContainer.addEventListener("contextmenu", function (){
                        console.log("click")
                        const node = args.node
                        const port = args.port
                        console.log("node,",node.id)
                        vue.singlePortFormVisible = true
                        vue.singlePort.nodeID = node.id
                        vue.singlePort.portID = port.id
                        vue.singlePort.bandwidth = port.bandwidth
                        vue.singlePort.delay = port.delay
                        // console.log(args.getPortByID(args.node.port.id))
                        let theport = node.getPort(port.id)
                        theport.data  = 200
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
            // console.log("getEdge",edges)
            // console.log("getSource",edges[0].getSource())
            // console.log("getSourceCell",edges[0].getSourceCell())
            // console.log("getSourceNode",edges[0].getSourceNode())
            // console.log("getSourcePortId",edges[0].getSourcePortId())
            const result = edges.map(edge => {
                const {id} = edge.id
                const edgeId = edge.getData().id || id
                const edgeName = edge.getData().name
                const edgeBandwidth = edge.getData().bandwidth
                const edgeDelay = edge.getData().delay
                return {
                    id: edge.id,
                    sourceNodeName: edge.getSourceNode().data.name,
                    targetNodeName: edge.getTargetNode().data.name,
                    sourcePortID: edge.getSourcePortId(),
                    targetPortID: edge.getTargetPortId(),
                    edgeId:edgeId,
                    edgeName:edgeName,
                    edgeBandwidth:edgeBandwidth,
                    edgeDelay:edgeDelay
                }
            })
            console.log("result",result)
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
                                // event:"port:click",
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
                        bandwidth:100,
                        delay:0
                    },
                ],
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
            graph.on("node:dblclick", e => {
                let cell = e.cell
                console.log("node")
                if (cell) {
                    this.nodeShowAttrConfig = true;
                    // console.log(cell.data)
                    // console.log(cell._isNode)
                    // console.log(cell._isEdge)
                    this.nodeFrmData = Object.assign(cell.data || {}, {
                        isNode: cell._isNode,
                        isEdge: cell._isEdge
                    });
                } else {
                    this.nodeShowAttrConfig = false;
                }
            })
            graph.on("edge:click", e => {
                let cell = e.cell
                if (cell) {
                    this.edgeShowAttrConfig = true;
                    this.edgeFrmData = Object.assign(cell.data || {}, {
                        isNode: cell._isNode,
                        isEdge: cell._isEdge
                    });
                } else {
                    this.edgeShowAttrConfig = false;
                }
            })
            // graph.on("port:click", (e) => {
            //     console.log("bbbbbbbbbb")
            //     console.log(e)
            //
            //     // let cell = e.cell
            //     this.singlePortFormVisible = true
            //     console.log(e.cell)
            //     // console.log(this)
            //     console.log("aaaaaaa")
            //     // console.log(cell)
            // })
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
                    this.getAllJson()
                    break
                case 'jsonToGraph':
                    this.jsonToGraph()
                    break
                default:
                    break
            }
        },
        confirmNode() {
            let graph = this.graph;
            let selectedCell = graph.getSelectedCells()[0];
            selectedCell.setData(this.nodeFrmData);
            // let style = selectedCell.style;
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
            this.nodeShowAttrConfig = false;
        },
        confirmEdge() {
            let graph = this.graph;
            let selectedCell = graph.getSelectedCells()[0];
            selectedCell.setData(this.edgeFrmData);
            // let style = selectedCell.style;
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
            this.edgeShowAttrConfig = false;
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
                    },
                    bandwidth:100,
                    delay:0
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
            this.edgeTableDialogVisible = true
            let allEdges = this.getEdge()
            this.allEdge = []
            console.log("allEdges",allEdges)
            for (let i = 0; i < allEdges.length; i++) {
                let temp = {
                    sourceNodeName: allEdges[i].sourceNodeName,
                    targetNodeName: allEdges[i].targetNodeName,
                    sourcePortID: allEdges[i].sourcePortID,
                    targetPortID: allEdges[i].targetPortID,
                    bandwidth: allEdges[i].edgeBandwidth,
                    delay: allEdges[i].edgeDelay,
                }
                this.allEdge.push(temp)
            }
            this.openEdgeReport()
        },
        viewSinglePort() {
            console.log("bb")
            this.singlePortFormVisible = true
        },
        nodeClose() {
            this.nodeShowAttrConfig = false;
        },
        edgeClose() {
            this.edgeShowAttrConfig = false;
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
        jsonToGraph() {
            console.log(zhenshiData.nodes[0])
            console.log(this.publicPath)
            console.log(`${this.publicPath}public/路由器.svg`)
            const graph = this.graph
            const getModelFromOriginData = () => {
                const model = {
                    nodes: [],
                    edges: [],
                }
                zhenshiData.nodes.forEach((item) => {
                    // console.log(item)
                    model.nodes?.push({
                        id: item.id,
                        shape: 'circle',
                        width: item.size>100?item.size:100,
                        height: item.size>100?item.size:100,
                        x: item.x,
                        y: item.y,
                        attrs: {
                            body: {
                                fill: '#5F95FF',
                                stroke: 'transparent',
                            },
                            text: {
                                'text': "Client"
                            },
                            image: {
                                'xlink:href': 'https://gw.alipayobjects.com/os/s/prod/antv/assets/image/logo-with-text-73b8a.svg',
                            }
                        },
                        ports: {
                            groups: {
                                po: {
                                    position: {
                                        name: "ellipseSpread",
                                        args: {
                                            start: 0
                                        },
                                    },
                                    // 'position': 'left',
                                    label: {
                                        position: "top"
                                    },
                                    attrs: {
                                        circle: {
                                            r: 10,
                                            magnet: true,
                                            stroke: "#31d0c6",
                                            strokeWidth: 2,
                                            fill: "#fff"
                                        }
                                    }
                                }
                            },
                            items:
                            item.ports.items.map(
                                function (sitem) {
                                    return {
                                        id: sitem.id,
                                        group: "po",
                                        attrs: {
                                            text: {
                                                text: String(sitem.id)
                                            }
                                        },
                                        bandwidth: sitem.bandwidth,
                                        delay: sitem.delay
                                    }
                                }
                            )
                        },
                    })
                })
                zhenshiData.edges.forEach((item) => {
                    model.edges?.push({
                        source: item.source,
                        target: item.target,
                        attrs: {
                            line: {
                                stroke: '#A2B1C3',
                                strokeWidth: 2,
                                targetMarker: null,
                            },
                        },
                    })
                })
                return model
            }

            const forceLayout = new ForceLayout({
                type: 'force',
                center: [369, 180],
                preventOverlap: true,
                collideStrength:1,
                alphaMin: 0.2,
                alphaDecay: 0.1,
                nodeSpacing:30,
                linkDistance: () => {
                    return 100
                },
                nodeStrength: () => {
                    return -10
                },
                edgeStrength: () => {
                    return 0.1
                },
                tick: () => {
                    const model = getModelFromOriginData(zhenshiData)
                    graph.fromJSON(model)
                },
            })
            forceLayout.layout(zhenshiData)
            console.log(getModelFromOriginData(zhenshiData))

            // const gridLayout = new GridLayout({
            //     type: 'grid',
            //     width: 738,
            //     height: 360,
            //     sortBy: 'label',
            //     rows: 3,
            //     cols: 7,
            //     nodeSize: [100, 100],
            // })
            //
            // const model = gridLayout.layout(zhenshiData)
            // graph.fromJSON(model)


            // console.log("graphtoJson",graphToJsonData)
            // let atemp = graph.parseJSON(graphToJsonData)
            // console.log("parseJson:",atemp)
            // let btemp = graph.fromJSON(graphToJsonData)
            // console.log("fromJSON:",btemp)

            // console.log("testData", testData)
            // let targetData = this.layout(testData)
            // console.log("targetData", targetData)
            // this.graph.fromJSON(targetData)
            // console.log("zhenshiData", testData)
            // let targetData = this.layout(zhenshiData)
            // console.log("targetData", targetData)
            // this.graph.fromJSON(targetData)
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
