
<template>
  <q-layout view="hHh lpR fFf">
    <q-header elevated class="bg-primary text-white" height-hint="98">
      <q-toolbar>
        <q-toolbar-title>
          Linguagens Formais e Autômatos - JSON to AFD
        </q-toolbar-title>
        <q-btn color="white" text-color="primary" icon="file_upload" @click="openModalJsonUpload" label="Importar JSON"/>
        <q-btn class="q-ml-sm" dense flat round icon="menu" @click="toggleRightDrawer" />
      </q-toolbar>
    </q-header>

    <q-drawer show-if-above v-model="rightDrawerOpen" side="right" bordered>
      <div class="flex column q-gutter-md q-pa-sm">
        <q-input
          @update:model-value="refreshGraph"
          filled
          v-model="alfabeto"
          label="Alfabeto"
          hint="Ex: A,B,C"
        />

        <q-input
          @update:model-value="refreshGraph"
          filled
          v-model="estados"
          label="Conjunto de estados"
          hint="Ex: 1,2,3"
          lazy-rules
          :rules="[ val => val && val.length > 0 || 'Digite um conjunto de estados']"
        />

        <q-input
          @update:model-value="refreshGraph"
          filled
          v-model="funcao"
          label="Função programa"
          hint="Ex: (1,A,2)(1,B,4)(2,A,1)"
          lazy-rules
          :rules="[ val => val && val.length > 0 || 'Digite uma função programa']"
        />

        <q-input
          @update:model-value="refreshGraph"
          filled
          v-model="estado_inicial"
          label="Estado inicial"
          hint="Ex: 1"
          lazy-rules
          :rules="[ val => val && val.length > 0 || 'Digite uma função programa']"
        />

        <q-input
          @update:model-value="refreshGraph"
          filled
          v-model="estado_final"
          label="Conjunto de estados finais"
          hint="Ex: 4"
          lazy-rules
          :rules="[ val => val && val.length > 0 || 'Digite uma função programa']"
        />

          <q-chip clickable :style="{backgroundColor: initialState, minWidth: '150px'}" text-color="white" icon-right="colorize">
            <template #default>
              <span>Cor do Estado Incial</span>
              <q-popup-proxy @update:model-value="refreshGraph" cover transition-show="scale" transition-hide="scale">
                <q-color v-model="initialState" />
              </q-popup-proxy>
            </template>
          </q-chip>

          <q-chip clickable :style="{backgroundColor: state, minWidth: '150px'}" text-color="white" icon-right="colorize">
            <template #default>
              <span>Cor dos Estados</span>
              <q-popup-proxy @update:model-value="refreshGraph" cover transition-show="scale" transition-hide="scale">
                <q-color v-model="state" />
              </q-popup-proxy>
            </template>
          </q-chip>

          <q-chip clickable :style="{backgroundColor: endState, minWidth: '150px'}" text-color="white" icon-right="colorize">
            <template #default>
              <span>Cor do Estado Final</span>
              <q-popup-proxy @update:model-value="refreshGraph" cover transition-show="scale" transition-hide="scale">
                <q-color v-model="endState" />
              </q-popup-proxy>
            </template>
          </q-chip>

        <q-toggle
          @update:model-value="refreshGraph"
          v-model="particles"
          color="pink"
          size="md"
          icon="more_horiz"
          label="Partículas"
        />
        <q-toggle
          @update:model-value="refreshGraph"
          v-model="focusNode"
          color="blue"
          size="md"
          icon="center_focus_strong"
          label="Focus Node"
        />
        <q-toggle
          @update:model-value="refreshGraph"
          v-model="textNode"
          color="green"
          size="md"
          checked-icon="sort_by_alpha"
          unchecked-icon="scatter_plot"
          label="Estado em texto"
        />
        <q-toggle
          @update:model-value="refreshGraph"
          v-model="mode"
          color="green"
          size="md"
          checked-icon="view_in_ar"
          unchecked-icon="account_tree"
          label="Modo de Visualização 2D/3D"
        />
      </div>
    </q-drawer>

    <q-page-container>
      <div v-if="mode" id="3d-graph"></div>
      <div v-else id="graph"></div>
    </q-page-container>

    <q-dialog ref="modal" v-model="openJsonUpload">
      <q-card class="flex column">
        <q-card-section class="flex flex-center q-pa-sm" style="background-color: #1976D2">
          <p style="font-size: 18px" class="text-white">Importar arquivo .JSON</p>
        </q-card-section>
        <q-card-section>
          <q-file filled accept=".json" label="Importar arquivo .JSON" v-model="jsonFile">
            <template v-slot:prepend>
              <q-icon name="file_upload" />
            </template>
          </q-file>
        </q-card-section>
        <q-card-section>
          <q-list bordered separator>
            <q-item-label header>
              Abaixo está o modelo para os tipos de arquivos aceitos.
              Basta clicar no modelo desejado e o download será iniciado.
            </q-item-label>
            <q-item  class="q-mt-sm" clickable v-ripple>
              <q-item-section avatar>
                <q-icon color="primary" name="file_download" />
              </q-item-section>

              <q-item-section>
                <q-item-label style="color: #1976D2">Modelo AFD .JSON</q-item-label>
              </q-item-section>

              <q-item-section side>
                <div class="row">
                  <q-icon @click="downloadModelo" style="cursor: pointer; padding-right: 5px"  size="sm" name="download" />
                </div>
              </q-item-section>
            </q-item>
          </q-list>
        </q-card-section>
        <q-separator/>
        <q-card-actions class="flex justify-between">
          <q-btn color="negative" @click="this.$refs.modal.hide()"  label="Cancelar"></q-btn>
          <q-btn color="positive" label="Enviar" @click="uploadFile"></q-btn>
        </q-card-actions>
      </q-card>
    </q-dialog>

  </q-layout>
</template>

<script>
import {defineComponent, ref} from 'vue'
import EssentialLink from 'components/EssentialLink.vue'
import ForceGraph3D from "3d-force-graph";
import SpriteText from "three-spritetext";
import {CSS2DRenderer} from "three-css2drender";
import * as THREE from 'three';
import {TextGeometry} from 'three/addons/geometries/TextGeometry.js';
import {FontLoader} from 'three/addons/loaders/FontLoader.js';
import ForceGraph from "force-graph";
import {useQuasar} from "quasar";
import modelo from "../../public/modelo.json"
import { saveAs } from 'file-saver';
export default defineComponent({
  name: 'MainLayout',

  components: {
    EssentialLink
  },

  methods: {
    downloadModelo(){
      console.log('aaa')
      const json = modelo;
      const jsonString = JSON.stringify(json, null, 2);
      const blob = new Blob([jsonString], { type: 'application/json' });
      saveAs(blob, 'modelo.json');
    },
    openModalJsonUpload() {
      this.openJsonUpload = true;
    },
    calcColor(afd, id) {
      if (afd.initialState == id) {
        return this.initialState;
      }

      if (afd.endState.includes(id)) {
        return this.endState;
      }

      return this.state;
    },
    getForm() {
      const alphabet = this.alfabeto.split(',');
      const allStates = this.estados.split(',');
      const programFunction = this.funcao.trim().split(/[()]/);
      const initialState = this.estado_inicial.split(',');
      const endState = this.estado_final.trim().split(',');
      console.log(programFunction)
      return {alphabet, allStates, programFunction, initialState, endState}
    },

    getAFD() {
      const data = this.getForm();

      const nodes = [];
      const links = [];

        // Criar os nós
      data.allStates.forEach(state => {
        nodes.push({
          id: state,
          color: this.calcColor(data, state)
        });
      });

      // Criar os links
      data.programFunction.forEach(func => {
        if (func) {
          const [source, label, target, curvature = 0.3] = func.split(",");
          links.push({
            source,
            target,
            curvature,
            label,
          });
        }
      });

      // Criar o objeto do grafo
      const graph = {
        nodes,
        links
      };

      return(graph)
    }
    ,
    getVariables() {
      const particles = this.particles;
      const focusNode = this.focusNode;
      const textNode = this.textNode;
      return {particles, focusNode, textNode}
    },
    refreshGraph() {
      setTimeout(() => {
        function getQuadraticXY3D(t, sx, sy, sz, cp1x, cp1y, cp1z, ex, ey, ez) {
          return {
            x: (1 - t) * (1 - t) * sx + 2 * (1 - t) * t * cp1x + t * t * ex,
            y: (1 - t) * (1 - t) * sy + 2 * (1 - t) * t * cp1y + t * t * ey,
            z: (1 - t) * (1 - t) * sz + 2 * (1 - t) * t * cp1z + t * t * ez
          };
        }

        function getQuadraticXY2D(t, sx, sy, cp1x, cp1y, ex, ey) {
          return {
            x: (1 - t) * (1 - t) * sx + 2 * (1 - t) * t * cp1x + t * t * ex,
            y: (1 - t) * (1 - t) * sy + 2 * (1 - t) * t * cp1y + t * t * ey,
          };
        }

        if (this.mode === true) {
          const variables = this.getVariables();

          const graph = this.getAFD();

          let Graph = ForceGraph3D({
            extraRenderers: [new CSS2DRenderer()]
          })
          (document.getElementById('3d-graph'))
          // .width(document.getElementById('container').offsetWidth)
          // .height(document.getElementById('container').offsetHeight)
          Graph.nodeThreeObject(node => {
            const geometry = new THREE.SphereGeometry(5, 22, 32);
            const material = new THREE.MeshLambertMaterial({
              color: node.color,
              transparent: true,
              opacity: 0.7
            });

            const sphere = new THREE.Mesh(geometry, material);

            const loader = new FontLoader();
            loader.load('https://threejs.org/examples/fonts/helvetiker_regular.typeface.json', function (font) {
              const textGeometry = new TextGeometry(node.id, {
                font: font,
                size: 3,
                height: 0.5,
                curveSegments: 12,
                bevelEnabled: false
              });

              const textMaterial = new THREE.MeshLambertMaterial({color: 0xffffff});
              const textMesh = new THREE.Mesh(textGeometry, textMaterial);

              // Centralizar o texto na esfera
              const sphereCenter = new THREE.Vector3();
              geometry.computeBoundingBox();
              geometry.boundingBox.getCenter(sphereCenter);

              const textBoundingBox = new THREE.Box3().setFromObject(textMesh);
              const textCenter = new THREE.Vector3();
              textBoundingBox.getCenter(textCenter);

              textMesh.position.copy(sphereCenter);
              textMesh.position.sub(textCenter);

              sphere.add(textMesh);
            });

            return sphere;
          })
            .graphData(graph)
            .linkLabel('label')
            .linkThreeObjectExtend(true)
            .linkThreeObject(link => {
              const sprite = new SpriteText(`${link.label}`);
              sprite.color = 'lightgrey';
              sprite.textHeight = 1.5;
              sprite.name = link;
              return sprite;
            })
            .linkPositionUpdate((sprite, {start, end}, link) => {

              let middlePos = getQuadraticXY3D(
                0.5,
                start.x,
                start.y,
                start.z,
                link.__curve.v1.x,
                link.__curve.v1.y,
                link.__curve.v1.z,
                end.x,
                end.y,
                end.z
              );

              // Position sprite
              Object.assign(sprite.position, middlePos);


            })
            .linkDirectionalArrowLength(2)
            .linkDirectionalArrowRelPos(1)
            .linkCurvature('curvature')
            .cameraPosition({z: 90})

          if (variables.particles) {
            Graph
              .linkDirectionalParticles("target")
              .linkDirectionalParticleSpeed(d => 0.01)
              .cameraPosition({z: 90})

          }
          if (variables.focusNode) {
            Graph.onNodeClick(node => {
              const distance = 40;
              const distRatio = 1 + distance / Math.hypot(node.x, node.y, node.z);

              const newPos = node.x || node.y || node.z
                ? {x: node.x * distRatio, y: node.y * distRatio, z: node.z * distRatio}
                : {x: 0, y: 0, z: distance}; // special case if node is in (0,0,0)

              Graph.cameraPosition(
                newPos, // new position
                node, // lookAt ({ x, y, z })
                3000  // ms transition duration
              );
            });
          }
          if (variables.textNode) {
            Graph.nodeThreeObject(node => {
              const sprite = new SpriteText(node.id);
              sprite.material.depthWrite = false; // make sprite background transparent
              sprite.color = node.color;
              sprite.textHeight = 8;
              return sprite;
            })
          }

        } else {
          const variables = this.getVariables();
          const graph = this.getAFD();

          const Graph = ForceGraph()
          (document.getElementById('graph'))
            .linkLabel('label')
            .linkAutoColorBy("#fff")
            .backgroundColor("#000")
            .nodeCanvasObject((node, ctx) => {
              ctx.beginPath();
              ctx.arc(node.x, node.y, 5, 0, 2 * Math.PI, false);
              ctx.fillStyle = node.color;
              ctx.fill();

              // Estilizar o texto
              ctx.fillStyle = 'white';
              ctx.font = 'bold 8px Arial';
              ctx.textAlign = 'center';
              ctx.textBaseline = 'middle';

              // Definir a letra a ser exibida
              const letter = node.id;

              // Posicionar o texto no centro do círculo
              ctx.fillText(letter, node.x, node.y);

            })
            .linkDirectionalArrowLength(2)
            .linkCurvature(0.3)
            .zoom(15, 50)
            .linkCanvasObjectMode(() => 'after')
            .linkCanvasObject((link, ctx) => {
              const MAX_FONT_SIZE = 4;
              const LABEL_NODE_MARGIN = Graph.nodeRelSize() * 1.5;

              const start = link.source;
              const end = link.target;

              // ignore unbound links
              if (typeof start !== 'object' || typeof end !== 'object') return;
              let middlePos = getQuadraticXY2D(
                0.5,
                start.x,
                start.y,
                link.__controlPoints[0],
                link.__controlPoints[1],
                end.x,
                end.y
              );

              const relLink = {x: end.x - start.x, y: end.y - start.y};

              const maxTextLength = Math.sqrt(Math.pow(relLink.x, 2) + Math.pow(relLink.y, 2)) - LABEL_NODE_MARGIN * 2;

              let textAngle = Math.atan2(relLink.y, relLink.x);
              // maintain label vertical orientation for legibility
              if (textAngle > Math.PI / 2) textAngle = -(Math.PI - textAngle);
              if (textAngle < -Math.PI / 2) textAngle = -(-Math.PI - textAngle);

              const label = `${link.label}`;

              // estimate fontSize to fit in link length
              ctx.font = '1px Sans-Serif';
              const fontSize = Math.min(MAX_FONT_SIZE, maxTextLength / ctx.measureText(label).width);
              ctx.font = `${fontSize}px Sans-Serif`;
              const textWidth = ctx.measureText(label).width;
              const bckgDimensions = [textWidth, fontSize].map(n => n + fontSize * 0.2); // some padding

              // draw text label (with background rect)
              ctx.save();
              ctx.translate(middlePos.x, middlePos.y);
              ctx.rotate(textAngle);

              ctx.fillStyle = 'rgba(0, 0, 0, 0)'; // Transparente
              ctx.fillRect(-bckgDimensions[0] / 2, -bckgDimensions[1] / 2, ...bckgDimensions);

              ctx.fillStyle = 'white'; // Branco
              ctx.textAlign = 'center';
              ctx.textBaseline = 'middle';
              ctx.fillText(label, 0, 0);
              ctx.restore();
            })
            .linkDirectionalArrowRelPos(1)
            .graphData(graph);

        }

      }, 0o100);
    },

    uploadFile() {

      const self = this;

      this.openJsonUpload = false;
      this.$q.notify('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA')
      const reader = new FileReader();
      reader.onload = (event) => {
        const content = event.target.result;
        const fromJson = JSON.parse(content);
        self.funcao = fromJson['function'] = fromJson['function'].map(item => item).join("");
        self.alfabeto = fromJson['alphabet'] = fromJson['alphabet'].map(item => item).join();
        self.estados = fromJson['states'] = fromJson['states'].map(item => item).join();
        self.estado_inicial = fromJson['start_state'];
        self.estado_final = fromJson['end_state'] = fromJson['end_state'].map(item => item).join();
        self.refreshGraph()
      }
      reader.readAsText(this.jsonFile);


    },
  },
  mounted() {
    this.refreshGraph(null);
  },
  setup () {
    const quasar = useQuasar()
    const rightDrawerOpen = ref(false)
    let alfabeto = ref("A,B");
    let estados = ref("1,2,3,4");
    let funcao = ref("(1,A,2)(1,B,4)(2,A,1)(2,B,3)(3,B,2)(3,A,4)(4,A,3)(4,B,1)");
    let estado_inicial = ref("1");
    let estado_final = ref("4");
    let initialState = ref("#0dd123");
    let state = ref("#043ec7");
    let endState = ref("#f70c0c");
    let particles = ref(false);
    let focusNode = ref(false);
    let textNode = ref(false);
    let mode = ref(true)
    let openJsonUpload = ref(false);
    let jsonFile = ref(null)
    let graphFromJson = ref(null)

    return {
      alfabeto: alfabeto,
      estados: estados,
      funcao: funcao,
      estado_inicial: estado_inicial,
      estado_final: estado_final,
      initialState: initialState,
      state: state,
      endState: endState,
      particles: particles,
      focusNode: focusNode,
      textNode: textNode,
      mode: mode,
      openJsonUpload: openJsonUpload,
      jsonFile: jsonFile,
      quasar: quasar,
      graphFromJson: graphFromJson,
      itemModelo: "../../public/modelo.json",
      rightDrawerOpen,
      toggleRightDrawer () {
        rightDrawerOpen.value = !rightDrawerOpen.value
      }
    }
  }
})
</script>
