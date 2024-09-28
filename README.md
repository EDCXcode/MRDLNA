
# MRDLNA
![](https://img.shields.io/badge/project-iOS-blue.svg)
![](https://img.shields.io/badge/install-CocoaPods-orange.svg)
![](https://img.shields.io/badge/LANG-ObjC-brightgreen.svg)
# Dependencies

- iOS DLNA Function 
- iOS DLNA 投屏功能, 支持各大主流电视盒子(小米,华为,乐视,移动魔百盒等), 可以播放,暂停,快进退,调音量,退出.
iOS 16.0后 实现投屏协议 需要向苹果申请多播权限
申请网站权限
https://developer.apple.com/contact/request/networking-multicast
参考文档
https://www.jianshu.com/p/f5379ba4c250
# Usage

```
pod 'MRDLNA', :git => 'https://github.com/EDCXcode/MRDLNA.git' 
```
    lazy var dlnaManager = {
        let dlna = MRDLNA.sharedMRDLNAManager()
        dlna?.delegate = self
        //搜索完成
        dlna?.blockSearch = {[weak self] falg in
         self?.searchDone()
        }
        //Socket关闭
        dlna?.blockSocketOff = {[weak self] in
            DispatchQueue.main.sync {
             //保证主线程刷新UI
            }
        }
        return dlna
    }()
     override func viewDidLoad() {
        super.viewDidLoad()
         //延迟开启搜索设备
         DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {[weak self] in
            self?.dlnaManager?.startSearch()
       }
    }


- Search Devices

```
<DLNADelegate>

- (void)searchDLNAResult:(NSArray *)devicesArray{
    NSLog(@"Find devices");
    //self.deviceArr = devicesArray;
    //[self.dlnaTable reloadData];
}

- (void)dlnaStartPlay{
    NSLog(@"DLNA Success Start Play");
}


```

- Play Control

```
@property(nonatomic,strong) MRDLNA *dlnaManager;

#pragma mark -Play Control

/**
 Quit
 */
- (IBAction)closeAction:(id)sender {
    [self.dlnaManager endDLNA];
}


/**
 Play/Pause
 */
- (IBAction)playOrPause:(id)sender {
    if (_isPlaying) {
        [self.dlnaManager dlnaPause];
    }else{
        [self.dlnaManager dlnaPlay];
    }
    _isPlaying = !_isPlaying;
}


/**
 SeekChange
 */
- (IBAction)seekChanged:(UISlider *)sender{
    NSInteger sec = sender.value * 60 * 60;
    NSLog(@"播放进度条======>: %zd",sec);
    [self.dlnaManager seekChanged:sec];
}

/**
 VolumeChange
 */
- (IBAction)volumeChange:(UISlider *)sender {
    NSString *vol = [NSString stringWithFormat:@"%.f",sender.value * 100];
    NSLog(@"音量========>: %@",vol);
    [self.dlnaManager volumeChanged:vol];
}


/**
 PlayNextMovie
 */
- (IBAction)playNext:(id)sender {
    NSString *testVideo = @"http://wvideo.spriteapp.cn/video/2016/0328/56f8ec01d9bfe_wpd.mp4";
    [self.dlnaManager playTheURL:testVideo];
}
```

# For more information please see demo

## License

WTFPL – Do What the Fuck You Want to Public License


