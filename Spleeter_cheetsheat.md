# Spleeter
Demix Audio Tracks into single STEMS/Tracks, based on a trained model.

[Spleeter Repository](https://github.com/deezer/spleeter.git)


### Conda Environment

Activate Spleeter

```
conda activate spleeter-cpu
```
Deactivate Conda (and so spleeter as well)

```
conda deactivate
```



### Separate Soundfiles

#### Show Options

```
spleeter separate -h
```



#### Separate with 2stems model

vocals / accompainment

```
spleeter separate -i input_path.wav -c wav -o output_folder_path
```

`-o` creates an output folder

`-c` Choose a output format from: wav, mp3, ogg, m4a, wma, flac



#### Separate with 4stems model

vocals / bass / drums / other

```
spleeter separate -i input_path.wav -c wav -o output_folder_path -p spleeter:4stems
```



#### Separate with 5stems model

vocals / bass / drums / piano / other

```
spleeter separate -i input_path.wav -c wav -o output_folder_path -p spleeter:5stems
```

