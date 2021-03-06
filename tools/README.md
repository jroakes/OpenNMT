# Tools

This directory contains additional tools.

## Tokenization/Detokenization

### Dependencies

* `bit32` for Lua < 5.2

### Tokenization
To tokenize a corpus:

```
th tools/tokenize.lua OPTIONS < file > file.tok
```

where the options are:

* `-mode`: can be `aggressive` or `conservative` (default). In conservative mode, letters, numbers and '_' are kept in sequence, hyphens are accepted as part of tokens. Finally inner characters `[.,]` are also accepted (url, numbers).
* `-sep_annotate`: if set, add reversible separator mark to indicate separator-less or BPE tokenization (preference on symbol, then number, then letter)
* `-case_feature`: generate case feature - and convert all tokens to lowercase
  * `N`: not defined (for instance tokens without case)
  * `L`: token is lowercased (opennmt)
  * `U`: token is uppercased (OPENNMT)
  * `C`: token is capitalized (Opennmt)
  * `M`: token case is mixed (OpenNMT)
* `-bpe_model`: when set, activate BPE using the BPE model filename

Note:

* `￨` is the feature separator symbol - if such character is used in source text, it is replace by its non presentation form `│`
* `￭` is the separator mark (generated in `-sep_annotate marker` mode) - if such character is used in source text, it is replace by its non presentation form `■`

### Detokenization

If you activate `sep_annotate` marker, the tokenization is reversible - just use:

```
th tools/detokenize.lua [-case_feature] < file.tok > file.detok
```

## Release model

After training a model on the GPU, you may want to release it to run on the CPU with the `release_model.lua` script.

```
th tools/release_model.lua -model model.t7 -gpuid 1
```

By default, it will create a `model_release.t7` file. See `th tools/release_model.lua -h` for advanced options.

## Translation Server

OpenNMT includes a translation server for running translate remotely. This also is an
easy way to use models from other languages such as Java and Python. 

The server uses the 0MQ for RPC. You can install 0MQ and the Lua bindings on Ubuntu by running:

```
sudo apt-get install libzmq-dev
luarocks install https://raw.github.com/Neopallium/lua-zmq/master/rockspecs/lua-zmq-scm-1.rockspec  ZEROMQ_LIBDIR=/usr/lib/x86_64-linux-gnu/ ZEROMQ_INCDIR=/usr/include
```

Also you will need to install the OpenNMT as a library.

```
luarocks make rocks/opennmt-scm-1.rockspec
```

The translation server can be run using any of the arguments from `translate.lua`. 

```
th tools/translation_server.lua -port ... -model ...
```

It runs as a message queue that takes in a JSON batch of src sentences. For example the following 5 lines of Python
code can be used to send a single sentence for translation.

```python
import zmq, sys, json
sock = zmq.Context().socket(zmq.REQ)
sock.connect("tcp://127.0.0.1:5556")
sock.send(json.dumps([{"src": " ".join(sys.argv[1:])}]))
print sock.recv()
```

For a longer example, see our <a href="http://github.com/OpenNMT/Server/">Python/Flask server</a> in development. 


## Embedding Conversion


### Dependencies

* `zlib` for Lua

### Conversion

Produces dictionary-reduced encoder and decoder embedding files in t7 format given word2vec, glove, or auto-loaded language embeddings.

```
th tools/embedding_convert.lua OPTIONS
```

examples:

* Autoload
```
th tools/embedding_convert.lua -auto_load en -dict_file data/demo/demo.src.dict -save_data data/demo/demo-src
```

* word2vec
```
th tools/embedding_convert.lua -embed_type word2vec -embed_file data/demo/GoogleNews-vectors-negative300.bin -dict_file data/demo/demo.src.dict -save_data data/demo/demo-src
```

where the options are:

* `-dict_file`: Path to the outputted dict file from preprocess.lua. Must be run separately for src and tgt dictionaries.
* `-embed_file`: Path to embedding file. Ignored if 'auto_lang' is used.
* `-save_data`: Output file path/label.
* `-auto_lang`: Wikipedia Language Code to autoload embeddings.
* `-embed_type`: `word2vec` (default) or `glove`. Ignored if 'auto_lang' is used.
* `-normalize`: `true` (default) or `false`. Boolean to normalize the word vectors, or not.
* `-report_every`: `100000` (default). Print stats every this many lines read from embedding file.

Notes and Attribution:
* Must be run separately for src and tgt dictionary files to produce enc and dec embedding files, respectively.
* 'auto_lang' feature is provided courtesy of Rami Al-Rfou through Polygot ( <a href="https://pypi.python.org/pypi/polyglot">Project</a> / <a href="http://www.aclweb.org/anthology/W13-3520">Paper</a> )
* Polygot Wikipedia Language Codes can be referenced <a href="https://sites.google.com/site/rmyeid/projects/polyglot">here</a>.
* Some code for bin conversion used with permission from Michael Rotman. <a href="https://github.com/rotmanmi/word2vec.torch">Original Code</a>. 
