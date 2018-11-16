# Overview

// this seems like a very relevant link, might need to do some C programming
// general understanding of audio https://trac.pjsip.org/repos/wiki/media-flow
// https://trac.pjsip.org/repos/wiki/Audio_Dev_API

// ideas:
// disable mic on the conference level: https://www.pjsip.org/pjmedia/docs/html/group__PJMEDIA__CONF.htm#gga95e1f5fd21e35d6317cc218844c2e0baa5f326e3af5c9ff506e3d2ae8ef00fa08
// * pjmedia_conf_create this is interresting, but seems to be by default configured without
//     initial device

/*
what did i come up with for now (looking at debug messages at log level 5)
cange the following files:

--------
file: coreaudio_dev.m
function: init factory or something (first one)
line: 292
old: cdi->info.input_count = 1;
new: cdi->info.input_count = 0;

function: ca_stream_set_cap
line: 1962
old:     ca_stream_get_cap(s, PJMEDIA_AUD_DEV_CAP_INPUT_LATENCY, &latency);
maybe remove this

------
file: pjsua_aud.c
function: `pjsua_set_snd_dev2`  this is the base function 
(called by ALL helpers for bidirectional / capture only / rec only)


file: sound_port.c
status = pjmedia_aud_dev_default_param(rec_id, &param.base);
if (status != PJ_SUCCESS)
return status;

// KIRILL
//param.base.dir = PJMEDIA_DIR_CAPTURE_PLAYBACK;
param.base.dir = PJMEDIA_DIR_PLAYBACK;

--------------------
attempt 1
file: pjmedia/src/pjmedia
function: `create_sound_port`
in here the conference options can be passed, we would want to
pass a options with PJMEDIA_CONF_NO_MIC

this is called in function `pjmedia_conf_create`

!!
file: pjsua-lib/pjsua_aud.c calls `pjmedia_conf_create` in `pjsua_aud_subsys_init`


--------------------------
attempt 2

file: pjsua-lib/pjsua_aud.c
function:  `open_snd_dev` 
idea: `pjsua_var.snd_mode & PJSUA_SND_DEV_SPEAKER_ONLY` needs to be 1, this will cause 
`pjmedia_snd_port_create_player` to be called instead of `pjmedia_snd_port_create2` (bidirectional)

==>

`pjsua_var.snd_mode` is only affected in `pjsua_set_snd_dev2(pjsua_snd_dev_mode snd_parm)` which sets the created sound device

we need snd_parm be set to `PJSUA_SND_DEV_SPEAKER_ONLY`


--------------------------
shared core audio
It uses Units to connect to hardware, 

(1) `ca_factory_create_stream` this is the main function that calls `create_audio_unit`
(1) decides what types of unit to create using 2d argument `pjmedia_aud_param::dir`
*/


// actually building from source:
// script to build from source [https://gist.github.com/graetzer/2be7c80b4dfadaa3ecb5a1794710ef31]
// but this is probably my best bet: (I would need to replace the url for the source to my own
// github repo of the modified library
// https://github.com/chebur/pjsip -- this totally work, and then just use the local cocoa pod

