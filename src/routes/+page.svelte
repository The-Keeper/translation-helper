<script lang="ts"> 
import { Marked } from '@ts-stack/markdown';
import Processor from 'asciidoctor';
const processor = new Processor();

import { saveAs } from 'file-saver';

let data = [{id: 0, orig: "original-1", trns: "translation-1"}, {id: "str", orig: "orig-2", trns: "trns-2"}]

enum FileFormat {
	AsciiDoc,
	markdown
}

let format: FileFormat = FileFormat.AsciiDoc; 

function exportToAsciiDoc(data) {
    return data.map(item => {
		let orig = item.orig + '';
		if (orig.trim().length == 0) {
			return '\r\n';
		}
		if (orig.startsWith('=')) {
			orig = '//' + orig
		} else {
			orig = '// ' + orig
		}
        return [orig, item.trns].join("\r\n");
    }).join("\r\n\r\n");
}

let files: FileList;
$: if (files) {
	// Note that `files` is of type `FileList`, not an Array:
	// https://developer.mozilla.org/en-US/docs/Web/API/FileList
	console.log(files);

	for (const file of files) {
		console.log(`${file.name}: ${file.size} bytes`);
	}
}

function handleChange(event) {
		if (files.length > 0) {
			let file = files[0];

			let reader = new FileReader();
			reader.onloadend = function (event) {
				let arrayBuffer = reader.result;
				let text = new TextDecoder('UTF-8').decode(arrayBuffer);
				loadDataFromText(text)
			};

			reader.readAsArrayBuffer(file);
		}
    }


function handleComment(line: string) {
	const comment_start_patterns = [ "//", "[//]:" ]
	let res = line;
	for (let c of comment_start_patterns) {
		if (line.startsWith(c))
			res = line.substring(c.length)
	}
	return res.trim()
}

function loadDataFromText(text: string) {
	let new_data = []

		let orig = [];
		let trns = [];

		let line_arr = text.split(/\r?\n/);
		for (let i=0;i<line_arr.length;i++) {
			let l = line_arr[i]
			if (l.startsWith("//") || l.startsWith("[//]:")) {
				orig.push(handleComment(l))
			} else if (!l.trim().length) {
				new_data.push({id: i, orig: orig.join('\r\n'), trns: trns.join('\r\n') })
				orig = []
				trns = []
			} else {
				trns.push(l)
			}
		}
		if (orig) {
			new_data.push({id: line_arr.length, orig: orig.join('\r\n'), trns: trns.join('\r\n') })
		}
		
	if (new_data) {
		data = new_data
	}  
}

function saveDataToFile() {
	let exported = ""
	if (format == FileFormat.AsciiDoc) {
		exported = exportToAsciiDoc(data);
	} 
	var blob = new Blob([exported], {type: "text/plain;charset=utf-8"});
	saveAs(blob, "asciidoc.adoc");
}

// $: pretty = JSON.stringify(data);

</script>

<div>
<label for="import">Upload a document:</label>
<input accept="text/adoc" bind:files type="file" name="import" on:change={handleChange} />
</div>

<div>
	<table>
		<col />
		<col class="original" />
		<col class="translation" />
		<col class="processed" />
		{#each data as { id, orig, trns }, i}
		<tr>
			<td>{i} [{id}]</td> <td>{orig}</td> <td><textarea class="translation" cols=40 rows="4" bind:value={trns}/></td> 
			<td>{@html processor.convert(trns)}</td>
		</tr>
		{/each}
</div>


<!-- <div>{pretty}</div> -->

<button on:click={saveDataToFile}>Save</button>

<div class="text-column"></div>
<style>
	.original {
		width: 25vw;
	}

	.translation {
		width: 25vw;
		height: 100%;
	}

	.processed {
		width: 100%;
	}


</style>