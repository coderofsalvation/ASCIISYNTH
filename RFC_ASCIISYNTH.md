%%%
Title = "ASCIISYNTH"
area = "Music"
workgroup = "Music"

[seriesInfo]
name = "ASCIISYNTH"
value = "draft-ASCIISYNTH-leonvankammen-00"
stream = "IETF"
status = "informational"

date = 2022-08-01 

[[author]]
initials="L.R."
surname="van Kammen"
fullname="L.R. van Kammen"

%%%

<!-- for annotated version see: https://raw.githubusercontent.com/ietf-tools/rfcxml-templates-and-schemas/main/draft-rfcxml-general-template-annotated-00.xml -->

.# Abstract

ASCIISYNTH is specification to (de)serialize synthesizer presets from/to ascii-strings which can be copy/pasted. 
The goal is twofold: 

* offer file-agnostic ways of sharing presets (using chat or forums or LLVM's e.g.).
* offer a way for samplers to backup the synthparameters in the samplename, which generated the sample 

{mainmatter}

# What is ASCIISYNTH

<img src="https://2wa.gitlab.io/asset/img/thumb_ASCIISYNTH.jpg" style="max-width:400px"/>

An ASCIISYNTH preset is basically a string like this:

`FMX1.2!!C!!!23!!!!vf23lk2A`
`GRANSYN2.12a4bcfeff00ac065a`

It's format consist of the following sequence of tokens:

`<synth identifier><version><parametervalues_using_hex_or_chars>`

For example:

* synth ID: `FMX`
* synth version: `1.2` (not mandatory but adviced)
* parameters: `!!C!!!23!!!!vf23lk2A`

> NOTE: the width of the Synth ID, Version, and parameters can be decided by the author.

# Workflow 

* a user tweaks synthesizer/sampler parameters
* the parameters get encoded (to ASCIISYNTH)
* the ASCIISYNTH name becomes the sample/instrument/preset-name 
* the software can now auto-initialize synth(parameters) by detecting supported ASCIISYNTH prefixes

# Encoding/Decoding the parameters

The Synth ID & Version are very easy to detect, and are verbatim (de)serialized.
The parameters however, are (de)serialized to their ascii decimal values.
These decimal values are offset by `33` to keep the characters easy copy/paste-table (below decimal ascii-value 33 are unprintable characters).

## Example 

`FMX1.2!"`

* synth ID: `FMX`
* synth version: `1.2`
* parameter 1: 0  (=`!`=ASCII decimal 33)
* parameter 2: 1  (=`"`=ASCII decimal 34 )

## Parameter resolution

The resolution on how much characters the host-application can reserve for a sample/instrument/preset-name.

* Small (a samplename of 20 max chars e.g.): use max parameter resolution is 93 (range `0..93`) because of the ASCII requirement (126-33=93)
* Bigger: just use hexadecimal values (synth X version 1 with one parametervalue 16 (=hex 10) becomes string `X110` e.g.)

# Example implementation

Milkytracker has ASCIISYNTH implemented, the gist is basically:

```
#define SYN_PREFIX_V1 "M1"                 // samplename 'M<version><params>' hints XM editors that sample was created with milkysynth <version> using <params> 
#define SYN_PREFIX_V2 "M2"                 //
#define SYN_PREFIX_CHARS 2                 // "M*"
#define SYN_PARAMS_MAX 32-SYN_PREFIX_CHARS // max samplechars (32) minus "M*" (32-2=30)     
#define SYN_OFFSET_CHAR 33                 // ASCIISYNTH spec: printable chars only 33..127 = 0..93 (allows ascii copy/paste of synthpresets)
#define SYN_PARAM_MAX_VALUE 93             // 93 printable chars

// handy macro to normalize ASCIISYNTH 0..93 range to 0..1 floats
#define SYN_PARAM_NORMALIZE(x) (1.0f/(float)SYN_PARAM_MAX_VALUE)*x

char* serialize() {
  char str[SYN_PREFIX_CHARS+SYN_PARAMS_MAX];
  sprintf(str,"%s",SYN_PREFIX_V2);           // always bump this to latest version
  for( int i = 0; i < SYN_PARAMS_MAX ; i++ ){ 
    str[ i + SYN_PREFIX_CHARS] = i < synth->nparams ? (int)synth->param[i].value + SYN_OFFSET_CHAR : SYN_OFFSET_CHAR; 
  }
  printf("synth: '%s'\n",str);
  return str;
}

bool deserialize( string preset ) {
  if( preset.startsWith(SYN_PREFIX_V1) || preset.startsWith(SYN_PREFIX_V2) ){ // detect synth version(s) 
    const char *str = preset.getStrBuffer();
    int ID = str[ SYN_PREFIX_CHARS ] - SYN_OFFSET_CHAR; 
    synth = &(synths[ID]); // replace current synth & initialize
    for( int i = 0; i < preset.length() && i < SYN_PARAMS_MAX; i++ ){ 
       setParam(i, str[ i + SYN_PREFIX_CHARS ] - SYN_OFFSET_CHAR ); 
    }
    return true;
  }else return false;
}

```

# Contact

* leonvankammen|gmail.com

# IANA Considerations

This document has no IANA actions.

# Acknowledgments

TODO acknowledge.
