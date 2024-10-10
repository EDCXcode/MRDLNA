
# MRDLNA
![](https://img.shields.io/badge/project-iOS-blue.svg)
![](https://img.shields.io/badge/install-CocoaPods-orange.svg)
![](https://img.shields.io/badge/LANG-ObjC-brightgreen.svg)
# Dependencies

- iOS DLNA Function 
- iOS DLNA 投屏功能, 支持各大主流电视盒子(小米,华为,乐视,移动魔百盒等), 可以播放,暂停,快进退,调音量,退出.
- iOS 16.0后 实现投屏协议 需要向苹果申请多播权限
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

- 苹果系统隔空播放

import AVKit
var player : AVPlayer?
var cmTime : CMTime?
 //隔空播放按钮
lazy var routeView : AVRoutePickerView = {
        let view  =  AVRoutePickerView.init(frame: .zero)
        view.backgroundColor = .clear
        view.subviews.forEach { subview in
            subview.removeFromSuperview()
        }
        let bgimg = UIImageView.init(image: UIImage(named: ""))
        view.addSubview(bgimg)
        bgimg.snp.makeConstraints { make in
            make.size.equalTo(bgimg.intrinsicContentSize)
            make.center.equalToSuperview()
        }
        view.delegate = self
        return view
}()

override func viewDidLoad() {
        super.viewDidLoad()
        player = AVPlayer(url: NSURL(string: "播放链接")! as URL)
        //监听播放
        NotificationCenter.default.addObserver(self, selector: #selector(handleRouteChange), name:AVAudioSession.routeChangeNotification, object:AVAudioSession.sharedInstance())
        //播放进度
        self.cmTime = CMTime(seconds: 1.0, preferredTimescale: CMTimeScale(NSEC_PER_SEC))
        self.player?.addPeriodicTimeObserver(forInterval:self.cmTime!, queue: DispatchQueue.main) { [weak self] time in
            let currentTime = CMTimeGetSeconds(time)
            let duration = CMTimeGetSeconds((self?.player?.currentItem?.duration)!)
            //更新UI
        }
        //快进
        //self.player?.seek(to: CMTime(seconds: Double(2.0), preferredTimescale: 1))
}

//MARK: AVRoutePickerViewDelegate
extension XXXXViewController:AVRoutePickerViewDelegate{
    func routePickerViewWillBeginPresentingRoutes(_ routePickerView: AVRoutePickerView)   {
        //print("AirPlay界面弹出时回调")
    }
    func routePickerViewDidEndPresentingRoutes(_ routePickerView: AVRoutePickerView) {
        //print("AirPlay界面结束时回调")
    }
    func routePickerView(_ routePickerView: AVRoutePickerView, didUpdateRoutes routes: [AVRoutePickerView]) {
        // 当可用路由更新时调用
    }
    func routePickerView(_ routePickerView: AVRoutePickerView, didPickRoute route: AVRoutePickerView) {
        // 当用户选择一个新的路由时调用
    }
    //监听回调
    @objc func handleRouteChange(){
        if activeAirplayOutputRouteName().isEmpty{
            self.player?.pause()
        }else{
            self.player?.play()
        }
    }
    //获取当前链接的播放的设备信息
    func activeAirplayOutputRouteName() ->(String){
        var portName :String = ""
        let currentRoute = audioSession.currentRoute
        currentRoute.outputs.forEach { outputPort in
            if outputPort.portType == .airPlay{
                portName = outputPort.portName
            }
        }
        return portName
    }
}



