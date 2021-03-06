# Tacotron2

An implementation of Tacotron2 (excluding WaveNet-vocoder) in TensorFlow.

2018-01-30 incompleted,

> .models , .utils , hparams.py , train.py

2018-02-04 incompleted,

> .tacotron2

2018-02-05 incompleted,

> .tacotron2-18.02.05

2018-02-06 complete !

> .tacotron2-18.02.06

## Attentions:
> /tacotron2-18.02.06/attentions.py

There are two styles of attention-mechanism. (1.LocationBasedAttention, 2.HybridAttention)

1.LocationBasedAttention

Calculate the score using (query, alignments).

    score = tf.reduce_sum(self.v * tf.tanh(processed_query + location_f), [2])

2.HybridAttention

Calculate the score using (processed-memory, query, alignments)

    score = tf.reduce_sum(self.v * tf.tanh(self.keys + processed_query + location_f), [2])
		
## MyAttentionWrapper:

> /tacotron2-18.02.06/attention_wrapper.py

Wrapping attention with LSTMs directly, not with such a 'prenet wrapper'.

So there are 6 steps
- Step 1: Passed through a 'Pre-net'.

 "The prediction from the previous time step is first passed through a small "pre-net" contatining 2 fully connected layers of 256 hidden ReLU units."

    inputs = dec_prenet(inputs, hp.dec_prenet_sizes, scope='decoder_prenet_layer')
- Step 2: Mix the `inputs` and previous step's `attention` output via `cell_input_fn`.

 "The pre-net output and attention context vector are concatenated and ..."

    cell_inputs = self._cell_input_fn(inputs, state.attention)
- Step 3: Call the wrapped `cell` with this input and its previous state.

 "...and passed through a stack of 2 uni-directional LSTM layers with 1024 units."

    LSTM_output, next_cell_state = self._cell(cell_inputs, state.cell_state)
- Step 4: Concat 'LSTM_output' with 'context_vector(attention)'

 "The concatenation of the LSTM output and the attention context vector is then..."

    concat_output_LSTM = tf.concat([LSTM_output, state.attention], axis=-1)
- Step 5: Concatted output projected through linear-transform.

 "...then projected through a linear transform to produce a prediction of the target spectrogram frame."

    cell_output = projection(concat_output_LSTM, hp.num_mels, scope='decoder_projection_layer')
- Step 6: Calculate the score, alignments and attention.


    
		attention, alignments = _compute_attention(attention_mechanism, cell_output, previous_alignments[i],  self._attention_layers[i] if self._attention_layers else None)

# 
So many references from https://github.com/Rayhane-mamah/Tacotron-2
