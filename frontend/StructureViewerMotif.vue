<template>
    <div class="structure-panel">
        <StructureViewerTooltip attach=".structure-panel" />
        <div class="structure-wrapper" ref="structurepanel">
            <table class="tmscore-panel" v-bind="tmPanelBindings">
            <tr>
                <td class="left-cell">idf-score:</td>
                <td class="right-cell">{{ alignments[0].idfscore }}</td>
            </tr>
            <tr>
                <td class="left-cell">RMSD:</td>
                <td class="right-cell">{{ alignments[0].rmsd  }}</td>
            </tr>
            </table>
            <StructureViewerToolbar
                :isFullscreen="isFullscreen"
                :isSpinning="isSpinning"
                @makeImage="handleMakeImage"
                @resetView="handleResetView"
                @toggleFullscreen="handleToggleFullscreen"
                @toggleSpin="handleToggleSpin"
                style="position: absolute; bottom: 8px;"
            />
            <div class="structure-viewer" ref="viewport"></div>
        </div>
    </div>
</template>

<script>
// import Alignment from './Alignment.vue'
import StructureViewerMixin from './StructureViewerMixin.vue';
import { pulchra } from 'pulchra-wasm';
import StructureViewer from './StructureViewer.vue';
import StructureViewerToolbar from './StructureViewerToolbar.vue';
import StructureViewerTooltip from './StructureViewerTooltip.vue';
import { makeMatrix4, makePositionMap, transformStructure } from './Utilities.js'
import Panel from './Panel.vue'
import {Stage, Shape, Selection, download, ColormakerRegistry, PdbWriter, Color, concatStructures, StructureComponent } from 'ngl'

const processPdb = (rawpdb) => {
    let outpdb = '';
    let data = '';
    let ext = 'pdb';
    outpdb = rawpdb.trimStart();
    if (data[0] == "#" || data.startsWith("data_")) {
        ext = 'cif';
        // NGL doesn't like AF3's _chem_comp entries
        outpdb = outpdb.replaceAll("_chem_comp.", "_chem_comp_SKIP_HACK.");
    } else {
        for (let line of outpdb.split('\n')) {
            if (line.startsWith("ATOM")) {
                let numCols = Math.max(0, 80 - line.length);
                let newLine = line + ' '.repeat(numCols) + '\n';
                data += newLine
            }
        }
        outpdb = data;
    }
    return [data, ext]
}

export default {
    name: "StructureViewerMotif",
    components: {
        StructureViewerToolbar,
        StructureViewerTooltip,
    },
    mixins: [
        StructureViewerMixin
    ],
    data: () => ({
        selection: null,
    }),
    props: {
        alignments: { type: Array, required: true, },
        lineLen: { type: Number, required: true, },
        queryPdb: {type: String, required: true, },

        qRepr: { type: String, default: "licorice" },
        tRepr: { type: String, default: "cartoon" },
        bgColorLight: {type: String, default: "white"},
        bgColorDark: {type: String, default: "#1E1E1E"},
        // autoViewTime: { type: Number, default: 100 },
    },
    computed: {
        // hasSelection() {
        //     return !this.structureHighlights.some(e => e !== null);
        // }
        bgColor() {
            return this.$vuetify.theme.dark ? this.bgColorDark : this.bgColorLight;
        },
        tmPanelBindings: function() {
            return (this.isFullscreen) ? { 'style': 'margin-top: 10px; font-size: 2em; line-height: 2em' } : {  }
        },
        stageParameters: function() {
            return {
                log: 'folddisco',
                backgroundColor: this.bgColor,
                transparent: true,
                clipNear: -1000,
                clipFar: 1000,
                fogFar: 1000,
                fogNear: -1000,
            }
        },
    },
    methods: {
        // setQuerySelection() {
        //     let repr = this.stage.getRepresentationsByName("queryStructure");
        //     if (!repr) return;
        //     let sele = this.querySele;
        //     repr.setSelection(sele);
        //     repr.list[0].parent.autoView(sele, this.autoViewTime);
        //     if (this.showQuery === 0) {
        //         this.stage.getRepresentationsByName("querySurface-1").setVisibility(false);
        //         this.stage.getRepresentationsByName("querySurface-2").setVisibility(false);
        //     } else if (this.showQuery === 1) {
        //         this.stage.getRepresentationsByName("querySurface-1").setVisibility(true);
        //         this.stage.getRepresentationsByName("querySurface-2").setVisibility(false);
        //     } else {
        //         this.stage.getRepresentationsByName("querySurface-1").setVisibility(true);
        //         this.stage.getRepresentationsByName("querySurface-2").setVisibility(true);
        //     }
        // },
        // getFirstResidueNumber(map, i) {
        //     let start = this.lineLen * (i - 1);
        //     while (map[start] === null) start--;
        //     return map[start];
        // },
        // getQueryRowStartPos(alnIndex, i) { return this.getFirstResidueNumber(this.queryMaps[alnIndex], i) },
        // getTargetRowStartPos(alnIndex, i) { return this.getFirstResidueNumber(this.targetMaps[alnIndex], i) },
        // setEmptyHighlight() {
        //     this.highlights = this.alignments.map(a => new Array(Math.ceil(a.qAln.length / this.lineLen)).fill([undefined, undefined]))
        // },
        // setEmptyStructureHighlight() {
        //     this.structureHighlights = new Array(this.alignments.length).fill(null);
        // },
        // clearAllSelection() {
        //     this.setEmptyHighlight();
        //     this.setEmptyStructureHighlight();
        // },
        // setAlignmentSelection(selections) {
        //     // array per alignment, then array per line in alignment
        //     this.setEmptyHighlight();
        //     for (let [ alnId, startLine, startOffset, endLine, endOffset, _ ] of selections) {
        //         for (let i = startLine; i <= endLine; i++) {
        //             if (i === startLine) {
        //                 this.highlights[alnId][i] = [startOffset, (i === endLine) ? endOffset : this.lineLen];
        //             } else if (i === endLine) {
        //                 this.highlights[alnId][i] = [0, endOffset];
        //             } else {
        //                 this.highlights[alnId][i] = [0, this.lineLen];
        //             }
        //         }
        //     }
        // },
        // onResidueSelectStart(event, alnIndex, lineNo) {
        //     this.isSelecting = true;
        //     document.querySelector(".alignment-wrapper-outer")
        //         .classList.add("inselection");
        // },
        // onResiduePointerUp(event, targetAlnIndex, targetLineNo) {
        //     if (!this.isSelecting) {
        //         // handle as click
        //         // this.highlights[targetAlnIndex].splice(targetLineNo - 1, 1, [undefined, undefined]);
        //         let a = this.alignments[targetAlnIndex];
        //         this.highlights.splice(targetAlnIndex, 1, new Array(Math.ceil(a.qAln.length / this.lineLen)).fill([undefined, undefined]));
        //         this.structureHighlights.splice(targetAlnIndex, 1, null);
        //         window.getSelection().removeAllRanges();
        //         return;
        //     }
        //     var selection = window.getSelection()
            
        //     // Get text and (sequence) starting position for each selected alignment
        //     let chunks = [];
        //     let chunk = "";
        //     let prevWrapper = null;
        //     let currWrapper = null;
        //     let lineNo = 0;
        //     let alnIndex = 0;
        //     let start = {};
        //     for (let i = 0; i < selection.rangeCount; i++) {
        //         let range = selection.getRangeAt(i);
        //         currWrapper = range.startContainer.parentElement.closest(".alignment-wrapper-inner");
        //         alnIndex = parseInt(currWrapper.id);
        //         lineNo = parseInt(range.startContainer.parentElement.closest(".line").id);
                
        //         // Start/end containers will either be:
        //         // #text  - Start/end inside a span, so calculate lengths of spans until that point
        //         // <span> - Start/end of entire span (e.g. multiline selection). Start = 0, end = line length
        //         let sc = range.startContainer;
        //         let ec = range.endContainer;
        //         let startOffset = (sc.nodeType === 3) ? range.startOffset + calculateOffset(sc.parentElement) : 0;
        //         let endOffset = (ec.nodeType === 3) ? range.endOffset + calculateOffset(ec.parentElement) : this.lineLen;
                
        //         // Test for new container (alignment), store starting line/offset & calculate position in sequence
        //         // If in the same alignment, extend sequence and update end line/offset
        //         if (!prevWrapper) {
        //             prevWrapper = currWrapper;
        //             let preText = range.startContainer.textContent.slice(0, range.startOffset);
        //             start = {
        //                 startLine: lineNo,
        //                 startOffset: startOffset,
        //                 seqStart: this.getTargetRowStartPos(alnIndex, lineNo) + startOffset - countCharacter(preText, '-')
        //             }
        //         } else if (currWrapper != prevWrapper) {
        //             chunks.push([parseInt(prevWrapper.id), start, chunk]);
        //             chunk = "";
        //             prevWrapper = currWrapper;
        //             let preText = range.startContainer.textContent.slice(0, startOffset);
        //             start = {
        //                 startLine: lineNo,
        //                 startOffset: startOffset,
        //                 seqStart: this.getTargetRowStartPos(alnIndex, lineNo) + startOffset - countCharacter(preText, '-')
        //             }
        //         }
        //         chunk += range.toString();
        //         start.endLine = lineNo;
        //         start.endOffset = endOffset;
        //     }
        //     chunks.push([parseInt(prevWrapper.id), start, chunk])

        //     // For structure: aln Id, start in sequence, selection length
        //     for (let [ alnId, { seqStart }, text ] of chunks) {
        //         this.structureHighlights.splice(alnId, 1, [seqStart, text.replace(/[-]/g, '').length]);
        //     }
            
        //     // For sequence: aln Id, line and start position (in start line), line and end position (in end line)
        //     this.setAlignmentSelection(chunks.map(([ alnId, { startLine, startOffset, endLine, endOffset }, chunk ]) => (
        //         [ alnId, startLine - 1, startOffset, endLine - 1, endOffset, chunk.length ]
        //     )));

        //     // Make everything else selectable again
        //     this.resetUserSelect();

        //     // Clear selection afterwards to prevent weird highlighting after inserting spans
        //     window.getSelection().removeAllRanges();
        // },
        resetUserSelect() {
            this.isSelecting = false;
            let noselects = document.querySelectorAll(".inselection");
            noselects.forEach(el => { el.classList.remove("inselection") });
        }
    },
    watch: {
        // 'alignment': function() {
        //     this.updateMaps()
        // }
    },
    // beforeMount() {
    //     // this.updateMaps() // What does this do?
    //     // this.setEmptyHighlight();
    //     // this.setEmptyStructureHighlight();
    // },
    async mounted() {
        if (typeof(this.alignments[0].tCa) == "undefined" || typeof(this.queryPdb) == "undefined")
            return;

        try {
            const ticket = this.$route.params.ticket;
            const match = this.alignments[0];
            const re = "api/result/folddisco/" + ticket + '?format=pdb&database=' + match.db +'&id=' + match.target;
            const request = await this.$axios.get(re,
                {headers: { // RACHEL: recover
                    'Cache-Control': 'no-cache'
                }}
            );
            targetPdb = request.data;
        } catch (e) {
            // throw e
            alert("Error: " + (e.response?.data || e.message || "Unknown"));
            return
        }
        let [queryPdb, qext] = processPdb(this.queryPdb);
        let [targetPdb, text] = processPdb(targetPdb);
        const query = await this.stage.loadFile(new Blob([queryPdb], { type: 'text/plain' }), { ext: qext, firstModelOnly: true, name: 'queryStructure'});
        const target = await this.stage.loadFile(new Blob([targetPdb], { type: 'text/plain' }), { ext: text, firstModelOnly: true, name: 'targetStructure'});

        const t = this.alignments[0].tmat.split(',').map(x => parseFloat(x));
        let u = this.alignments[0].umat.split(',').map(x => parseFloat(x));
        u = [
            [u[0], u[1], u[2]],
            [u[3], u[4], u[5]],
            [u[6], u[7], u[8]],
        ];
        const matrix = makeMatrix4(t, u);
        query.setTransform(matrix);

        query.addRepresentation(this.qRepr, {color: 'blue', name: "queryStructure"})
        target.addRepresentation(this.tRepr, {color: 'red', name: "targetStructure"}) 

        query.autoView();
    }
}
</script>

<style scoped>

.structure-panel {
    width: 100%;
    height: 100%;
    position: relative;
}
.structure-wrapper {
    width: 500px;
    height: 400px;
    margin: 0 auto;
}

.structure-viewer {
    width: 100%;
    height: 100%;
    overflow: hidden;
    padding: 0;
}

.structure-viewer canvas {
    border-radius: 2px;
}

/* @media screen and (max-width: 960px) {
    .structure-panel  {
        display: flex;
    }
    .structure-panel {
        flex-direction: column-reverse;
    }
    .structure-wrapper {
        padding-bottom: 1em;
    }
    .structure-wrapper {
        align-self: center;
    }
}

@media screen and (min-width: 961px) {
    .structure-wrapper {
        padding-left: 2em;
    }
} */

</style>
