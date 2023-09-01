<!DOCTYPE html>
<html>
<head>
  
  <meta name="GENERATOR" content="github.com/mmarkdown/mmark Mmark Markdown Processor - mmark.miek.nl">
  <meta charset="utf-8">
</head>
<body>

<!-- for annotated version see: https://raw.githubusercontent.com/ietf-tools/rfcxml-templates-and-schemas/main/draft-rfcxml-general-template-annotated-00.xml -->

<h1 class="special" id="abstract">Abstract</h1>

<p>ASCIISYNTH is specification to (de)serialize synthesizer presets from/to ascii-strings which can be copy/pasted.
The goal is twofold:</p>

<ul>
<li>offer file-agnostic ways of sharing presets (using chat or forums or LLVM&rsquo;s e.g.).</li>
<li>offer a way for samplers to backup the synthparameters in the samplename, which generated the sample</li>
</ul>
<section data-matter="main">
<h1 id="what-is-asciisynth">What is ASCIISYNTH</h1>

<p><img src="https://2wa.gitlab.io/asset/img/thumb_ASCIISYNTH.jpg" style="max-width:400px"/></p>

<p>An ASCIISYNTH preset is basically a string like this:</p>

<p><code>FMX1.2!!C!!!23!!!!vf23lk2A</code>
<code>GRANSYN2.12a4bcfeff00ac065a</code></p>

<p>It&rsquo;s format consist of the following sequence of tokens:</p>

<p><code>&lt;synth identifier&gt;&lt;version&gt;&lt;parametervalues_using_hex_or_chars&gt;</code></p>

<p>For example:</p>

<ul>
<li>synth ID: <code>FMX</code></li>
<li>synth version: <code>1.2</code> (not mandatory but adviced)</li>
<li>parameters: <code>!!C!!!23!!!!vf23lk2A</code></li>
</ul>

<blockquote>
<p>NOTE: the width of the Synth ID, Version, and parameters can be decided by the author.</p>
</blockquote>

<h1 id="workflow">Workflow</h1>

<ul>
<li>a user tweaks synthesizer/sampler parameters</li>
<li>the parameters get encoded (to ASCIISYNTH)</li>
<li>the ASCIISYNTH name becomes the sample/instrument/preset-name</li>
<li>the software can now auto-initialize synth(parameters) by detecting supported ASCIISYNTH prefixes</li>
</ul>

<h1 id="encoding-decoding-the-parameters">Encoding/Decoding the parameters</h1>

<p>The Synth ID &amp; Version are very easy to detect, and are verbatim (de)serialized.
The parameters however, are (de)serialized to their ascii decimal values.
These decimal values are offset by <code>33</code> to keep the characters easy copy/paste-table (below decimal ascii-value 33 are unprintable characters).</p>

<h2 id="example">Example</h2>

<p><code>FMX1.2!&quot;</code></p>

<ul>
<li>synth ID: <code>FMX</code></li>
<li>synth version: <code>1.2</code></li>
<li>parameter 1: 0  (=<code>!</code>=ASCII decimal 33)</li>
<li>parameter 2: 1  (=<code>&quot;</code>=ASCII decimal 34 )</li>
</ul>

<h2 id="parameter-resolution">Parameter resolution</h2>

<p>The resolution on how much characters the host-application can reserve for a sample/instrument/preset-name.</p>

<ul>
<li>Small (a samplename of 20 max chars e.g.): use max parameter resolution is 93 (range <code>0..93</code>) because of the ASCII requirement (126-33=93)</li>
<li>Bigger: just use hexadecimal values (synth X version 1 with one parametervalue 16 (=hex 10) becomes string <code>X110</code> e.g.)</li>
</ul>

<h1 id="example-implementation">Example implementation</h1>

<p>Milkytracker has ASCIISYNTH implemented, the gist is basically:</p>

<pre><code>#define SYN_PREFIX_V1 &quot;M1&quot;                 // samplename 'M&lt;version&gt;&lt;params&gt;' hints XM editors that sample was created with milkysynth &lt;version&gt; using &lt;params&gt; 
#define SYN_PREFIX_V2 &quot;M2&quot;                 //
#define SYN_PREFIX_CHARS 2                 // &quot;M*&quot;
#define SYN_PARAMS_MAX 32-SYN_PREFIX_CHARS // max samplechars (32) minus &quot;M*&quot; (32-2=30)     
#define SYN_OFFSET_CHAR 33                 // ASCIISYNTH spec: printable chars only 33..127 = 0..93 (allows ascii copy/paste of synthpresets)
#define SYN_PARAM_MAX_VALUE 93             // 93 printable chars

// handy macro to normalize ASCIISYNTH 0..93 range to 0..1 floats
#define SYN_PARAM_NORMALIZE(x) (1.0f/(float)SYN_PARAM_MAX_VALUE)*x

char* serialize() {
  char str[SYN_PREFIX_CHARS+SYN_PARAMS_MAX];
  sprintf(str,&quot;%s&quot;,SYN_PREFIX_V2);           // always bump this to latest version
  for( int i = 0; i &lt; SYN_PARAMS_MAX ; i++ ){ 
    str[ i + SYN_PREFIX_CHARS] = i &lt; synth-&gt;nparams ? (int)synth-&gt;param[i].value + SYN_OFFSET_CHAR : SYN_OFFSET_CHAR; 
  }
  printf(&quot;synth: '%s'\n&quot;,str);
  return str;
}

bool deserialize( string preset ) {
  if( preset.startsWith(SYN_PREFIX_V1) || preset.startsWith(SYN_PREFIX_V2) ){ // detect synth version(s) 
    const char *str = preset.getStrBuffer();
    int ID = str[ SYN_PREFIX_CHARS ] - SYN_OFFSET_CHAR; 
    synth = &amp;(synths[ID]); // replace current synth &amp; initialize
    for( int i = 0; i &lt; preset.length() &amp;&amp; i &lt; SYN_PARAMS_MAX; i++ ){ 
       setParam(i, str[ i + SYN_PREFIX_CHARS ] - SYN_OFFSET_CHAR ); 
    }
    return true;
  }else return false;
}

</code></pre>

<h1 id="contact">Contact</h1>

<ul>
<li>leonvankammen|gmail.com</li>
</ul>

<h1 id="iana-considerations">IANA Considerations</h1>

<p>This document has no IANA actions.</p>

<h1 id="acknowledgments">Acknowledgments</h1>

<p>TODO acknowledge.</p>
</section>

</body>
</html>

