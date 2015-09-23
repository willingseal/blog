#How to use DOUAudioStreamer

DOUAudioStreamer address:[https://github.com/douban/DOUAudioStreamer](https://github.com/douban/DOUAudioStreamer)

DOUAudioStreamer is a Core Audio based streaming audio player for iOS/Mac.

Here simple brief of its use, the text code from the DOUAudioStreamer project.

##Implement DOUAudioFile Protocol

By implementing DOUAudioFile protocol, construct an audio file can be a local file or a network file.

Track is a class below the protocol implementation.

<pre>
Track *track = [[Track alloc] init];
          [track setArtist:@"bob dylan"];
          [track setTitle:@"like a rolling stone"];
          NSURL *modelURL1 = [[NSBundle mainBundle] URLForResource:@"like a rolling stone" withExtension:@"mp3"];
          NSURL *modelURL2 = [NSURL URLWithString:@"http://mr7.doubanio.com/2867d1b829cddffa78318cdb7a3b34ce/1/fm/song/p616953_128k.mp4"]
          [track setAudioFileURL:modelURL1];
</pre>

##DOUAudioStreamer
DOUAudioStreamer is an audio player class
<pre>
DOUAudioStreamer *streamer = [DOUAudioStreamer streamerWithAudioFile:track];

if ([streamer status] == DOUAudioStreamerPaused || [streamer status] == DOUAudioStreamerIdle) {
        [streamer play];
    }else{
        [streamer pause];
    }
</pre>


KVO can listen DOUAudioStreamer by some of the properties, including the status, duration, bufferingRatio etc.
<pre>
[_streamer addObserver:self forKeyPath:@"status" options:NSKeyValueObservingOptionNew context:kStatusKVOKey];
        [_streamer addObserver:self forKeyPath:@"duration" options:NSKeyValueObservingOptionNew context:kDurationKVOKey];
        [_streamer addObserver:self forKeyPath:@"bufferingRatio" options:NSKeyValueObservingOptionNew context:kBufferingRatioKVOKey];
</pre>



Implement a callback
<pre>
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
  if (context == kStatusKVOKey) {
    [self performSelector:@selector(_updateStatus)
                 onThread:[NSThread mainThread]
               withObject:nil
            waitUntilDone:NO];
  }
  else if (context == kDurationKVOKey) {
    [self performSelector:@selector(_timerAction:)
                 onThread:[NSThread mainThread]
               withObject:nil
            waitUntilDone:NO];
  }
  else if (context == kBufferingRatioKVOKey) {
    [self performSelector:@selector(_updateBufferingStatus)
                 onThread:[NSThread mainThread]
               withObject:nil
            waitUntilDone:NO];
  }
  else {
    [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
  }
}

</pre>




Observe the state of the music player
<pre>

- (void)_updateStatus
{
  switch ([_streamer status]) {
  case DOUAudioStreamerPlaying:
    break;

  case DOUAudioStreamerPaused:
    break;

  case DOUAudioStreamerIdle:
    break;

  case DOUAudioStreamerFinished:
    break;

  case DOUAudioStreamerBuffering:
    break;

  case DOUAudioStreamerError:
    break;
  }
}

</pre>

Observe the progress of the music player

<pre>
- (void)_timerAction:(id)timer
{
  if ([_streamer duration] == 0.0) {
    [_progressSlider setValue:0.0f animated:NO];
  }
  else {
    [_progressSlider setValue:[_streamer currentTime] / [_streamer duration] animated:YES];
  }
}
</pre>


Buffer ratio observed music files, mainly network file
<pre>

- (void)_updateBufferingStatus
{
  [_miscLabel setText:[NSString stringWithFormat:@"Received %.2f/%.2f MB (%.2f %%), Speed %.2f MB/s", (double)[_streamer receivedLength] / 1024 / 1024, (double)[_streamer expectedLength] / 1024 / 1024, [_streamer bufferingRatio] * 100.0, (double)[_streamer downloadSpeed] / 1024 / 1024]];

  if ([_streamer bufferingRatio] >= 1.0) {
    NSLog(@"sha256: %@", [_streamer sha256]);
  }
}
</pre>

##DOUAudioVisualizer
By DOUAudioVisualizer class can display a visual music beat the view.
<pre>
	DOUAudioVisualizer *audioVisualizerView=[[DOUAudioVisualizer alloc] initWithFrame:CGRectMake((ScreenWidth-250)/2, 140, 250, 250)];
	[self.view addSubview:self.audioVisualizerView];
</pre>

    
    
    
    
    
    
    
    
    
    