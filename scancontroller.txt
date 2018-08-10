//
//  ViewController.swift
//  FoodVision
//
//  Created by Niranjan Senthilkumar on 6/6/18.
//  Copyright © 2018 NJ. All rights reserved.
//

import UIKit
import CoreML
import Vision
import AVFoundation
import CoreData

var postFoodImage = UIImageView(image: #imageLiteral(resourceName: "placeholderimage"))


class ScanController: UIViewController, FrameExtractorDelegate, AVCapturePhotoCaptureDelegate {
    
    var frameExtractor: FrameExtractor!
    
    private let session = AVCaptureSession()
    
    let scanArea: UIImageView = {
        let label = UIImageView()
        label.image = #imageLiteral(resourceName: "rectangle")
        label.alpha = 0.0
        return label
    }()
    
    let scanAreaButton: UIButton = {
        let button = UIButton(type: .system)
        button.addTarget(self, action: #selector(handleTap), for: .touchUpInside)
        button.tintColor = .clear
        return button
    }()
    
    let previewImage: UIImageView = {
        let image = UIImageView()
        image.backgroundColor = .black
        return image
    }()
    
    let iSee: UILabel = {
        let label = UILabel()
        label.text = "Detecting: no food"
        label.font = UIFont(name: "Avenir-Medium", size: 12)
        label.textColor = .white
        return label
    }()
    
    let cameraButton: UIButton = {
        let button = UIButton(type: .system)
        button.setImage(#imageLiteral(resourceName: "camera").withRenderingMode(.alwaysOriginal), for: .normal)
        button.addTarget(self, action: #selector(handleScan), for: .touchUpInside)
        return button
    }()
    
    let galleryButton: UIButton = {
        let button = UIButton(type: .system)
        button.setImage(#imageLiteral(resourceName: "galleryicon2").withRenderingMode(.alwaysOriginal), for: .normal)
        button.addTarget(self, action: #selector(handleTouch), for: .touchUpInside)
        button.addSubview(postFoodImage)
        postFoodImage.contentMode = .scaleAspectFill
        postFoodImage.clipsToBounds = true
        postFoodImage.anchor(top: button.topAnchor, left: button.leftAnchor, bottom: button.bottomAnchor, right: button.rightAnchor, paddingTop: 2, paddingLeft: 2, paddingBottom: 2, paddingRight: 2, width: 0, height: 0)
        return button
    }()
    
    let statsButton: UIButton = {
        let button = UIButton(type: .system)
        button.setImage(#imageLiteral(resourceName: "piechart2x").withRenderingMode(.alwaysOriginal), for: .normal)
        button.addTarget(self, action: #selector(handleStats), for: .touchUpInside)
        return button
    }()
    
    @objc func handleStats(){
        let pastVC = StatsController(collectionViewLayout: UICollectionViewFlowLayout())
        
        let transition = CATransition()
        transition.duration = 0.4
        transition.type = kCATransitionPush
        transition.subtype = kCATransitionFromTop
        transition.timingFunction = CAMediaTimingFunction(name:kCAMediaTimingFunctionEaseInEaseOut)
        view.window!.layer.add(transition, forKey: kCATransition)
        
        pastVC.navigationItem.titleView = UIImageView(image: #imageLiteral(resourceName: "Nutrition"))
        navigationController?.navigationBar.isTranslucent = false
        
        self.navigationController?.pushViewController(pastVC, animated: false)
    }
    
    
    @objc func handleTouch(){
        let pastVC = PastFoodController(collectionViewLayout: UICollectionViewFlowLayout())
        
        let transition = CATransition()
        transition.duration = 0.4
        transition.type = kCATransitionPush
        transition.subtype = kCATransitionFromLeft
        transition.timingFunction = CAMediaTimingFunction(name:kCAMediaTimingFunctionEaseInEaseOut)
        view.window!.layer.add(transition, forKey: kCATransition)
        
        pastVC.navigationItem.titleView = UIImageView(image: #imageLiteral(resourceName: "Foods Eaten"))
        navigationController?.navigationBar.isTranslucent = false
            
        self.navigationController?.pushViewController(pastVC, animated: false)
    }
    
    override func viewWillAppear(_ animated: Bool) {
        navigationController?.navigationBar.setBackgroundImage(UIImage(), for: UIBarMetrics.default)
        navigationController?.navigationBar.shadowImage = UIImage()
        navigationController?.navigationBar.isTranslucent = true
        navigationController?.view.backgroundColor = UIColor.clear
    }

    
    @objc func handleScan(){
        if iSee.text != "Detecting: no food" {
        
        let nutritionVC = NutritionController(collectionViewLayout: UICollectionViewFlowLayout())
        
        nutritionVC.navigationItem.titleView = UIImageView(image: #imageLiteral(resourceName: "FoodVision"))
        navigationController?.setNavigationBarHidden(false, animated: false)
        
        navigationController?.navigationBar.barTintColor = UIColor.backgroundPink
        
            
        navigationController?.navigationBar.isTranslucent = true
        navigationController?.navigationBar.setBackgroundImage(UIImage(), for: .default)
        navigationController?.navigationBar.shadowImage = UIImage()
        
        guard let scanT = self.iSee.text?.substring(from: 11) else {return}
        
        nutritionVC.scannedText = scanT
        
        let trimmedString = scanT.replacingOccurrences(of: " ", with: "")
        
        textScan = trimmedString
        
        let date = Date().timeIntervalSince1970

        self.scanArea.alpha = 0
            
        let postImage = previewImage.image
            
        nutritionVC.image = previewImage.image!
        nutritionVC.timeInterval = date
            
        var device : AVCaptureDevice!
            
        let settings = AVCapturePhotoSettings()
        let previewPixelType = settings.availablePreviewPhotoPixelFormatTypes.first!
        let previewFormat = [kCVPixelBufferPixelFormatTypeKey as String: previewPixelType,
                             kCVPixelBufferWidthKey as String: 160,
                             kCVPixelBufferHeightKey as String: 160]
        settings.previewPhotoFormat = previewFormat
            
        if #available(iOS 10.0, *) {
            let videoDeviceDiscoverySession = AVCaptureDevice.DiscoverySession(deviceTypes: [.builtInWideAngleCamera, .builtInDuoCamera], mediaType: AVMediaType.video, position: .unspecified)
            let devices = videoDeviceDiscoverySession.devices
            device = devices.first!

        } else {
            // Fallback on earlier versions
            device = AVCaptureDevice.default(for: AVMediaType.video)
        }

        do{
            if (device.hasTorch){
                try device.lockForConfiguration()
                device.torchMode = .off
                device.flashMode = .off
                device.unlockForConfiguration()
            }
        }catch{
            //DISABEL FLASH BUTTON HERE IF ERROR
        }
        
        navigationController?.pushViewController(nutritionVC, animated: true)
    
        }
    }
    
    
    @objc func handleInfo(){
        let infoVC = InfoController()

        self.present(infoVC, animated: true, completion: nil)
    }
    
    let detectLabel: UILabel = {
        let label = UILabel()
        label.backgroundColor = .clear
        return label
    }()
    
    var settingImage = false
    
    var currentImage: CIImage? {
        didSet {
            if let image = currentImage{
                self.detectScene(image: image)
            }
        }
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()

        
        frameExtractor = FrameExtractor()
        frameExtractor.delegate = self
        
        view.addSubview(previewImage)
        previewImage.anchor(top: view.topAnchor, left: view.leftAnchor, bottom: view.bottomAnchor, right: view.rightAnchor, paddingTop: 0, paddingLeft: 0, paddingBottom: 0, paddingRight: 0, width: 0, height: 0)

        
        view.addSubview(cameraButton)
        cameraButton.anchor(top: nil, left: nil, bottom: view.bottomAnchor, right: nil, paddingTop: 589, paddingLeft: 0, paddingBottom: 25, paddingRight: 0, width: 90, height: 90)
        cameraButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
        
        view.addSubview(galleryButton)
        galleryButton.anchor(top: nil, left: view.leftAnchor, bottom: nil, right: nil, paddingTop: 0, paddingLeft: 25, paddingBottom: 0, paddingRight: 0, width: 40, height: 40)
        galleryButton.centerYAnchor.constraint(equalTo: cameraButton.centerYAnchor).isActive = true
        
        view.addSubview(statsButton)
        statsButton.anchor(top: nil, left: nil, bottom: nil, right: view.rightAnchor, paddingTop: 0, paddingLeft: 0, paddingBottom: 0, paddingRight: 25, width: 40, height: 40)
        statsButton.centerYAnchor.constraint(equalTo: cameraButton.centerYAnchor).isActive = true
        
        view.addSubview(detectLabel)
        detectLabel.anchor(top: view.topAnchor, left: view.leftAnchor, bottom: view.bottomAnchor, right: view.rightAnchor, paddingTop: 0, paddingLeft: 0, paddingBottom: 616, paddingRight: 0, width: 0, height: 40)
        
        detectLabel.addSubview(iSee)
        iSee.anchor(top: detectLabel.topAnchor, left: nil, bottom: detectLabel.bottomAnchor, right: nil, paddingTop: 30, paddingLeft: 0, paddingBottom: 0, paddingRight: 0, width: 0, height: 0)
        iSee.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
        
        
        view.addSubview(scanAreaButton)
        scanAreaButton.addSubview(scanArea)
        scanAreaButton.anchor(top: view.topAnchor, left: view.leftAnchor, bottom: cameraButton.topAnchor, right: view.rightAnchor, paddingTop: 0, paddingLeft: 12, paddingBottom: 0, paddingRight: 12, width: 0, height: 270)
        scanArea.anchor(top: scanAreaButton.topAnchor, left: scanAreaButton.leftAnchor, bottom: scanAreaButton.bottomAnchor, right: scanAreaButton.rightAnchor, paddingTop: 161, paddingLeft: 0, paddingBottom: 108, paddingRight: 0, width: 0, height: 0)
        
        navigationItem.rightBarButtonItem = UIBarButtonItem(image: #imageLiteral(resourceName: "info2xwhite").withRenderingMode(.alwaysOriginal), style: .plain, target: self, action: #selector(handleInfo))

        navigationItem.leftBarButtonItem = UIBarButtonItem(image: #imageLiteral(resourceName: "flash").withRenderingMode(.alwaysOriginal), style: .plain, target: self, action: #selector(toggleFlash))
        
        }
    
    
    
    @objc func handleTap(){
        if (self.scanArea.alpha == 0.0){
            UIView.animate(withDuration: 0.4) {
                self.scanArea.alpha = 1.0
            }
        }
        else {
            UIView.animate(withDuration: 0.4) {
                self.scanArea.alpha = 0.0
            }
        }
    }
    
    @objc func toggleFlash() {
        var device : AVCaptureDevice!
        
        if #available(iOS 10.0, *) {
            let videoDeviceDiscoverySession = AVCaptureDevice.DiscoverySession(deviceTypes: [.builtInWideAngleCamera, .builtInDuoCamera], mediaType: AVMediaType.video, position: .unspecified)
            let devices = videoDeviceDiscoverySession.devices
            device = devices.first!
            
        } else {
            // Fallback on earlier versions
            device = AVCaptureDevice.default(for: AVMediaType.video)
        }
        
        if ((device as AnyObject).hasMediaType(AVMediaType.video))
        {
            if (device.hasTorch)
            {
                self.session.beginConfiguration()
                //self.objOverlayView.disableCenterCameraBtn();
                if device.isTorchActive == false {
                    self.flashOn(device: device)
                } else {
                    self.flashOff(device: device);
                }
                //self.objOverlayView.enableCenterCameraBtn();
                self.session.commitConfiguration()
            }
            
        }
        
    }
    
    
    func flashOn(device:AVCaptureDevice)
    {
        do{
            if (device.hasTorch)
            {
                try device.lockForConfiguration()
                device.torchMode = .on
                device.flashMode = .on
                device.unlockForConfiguration()
            }
        }catch{
            //DISABEL FLASH BUTTON HERE IF ERROR
        }
    }
    
    func flashOff(device:AVCaptureDevice)
    {
        do{
            if (device.hasTorch){
                try device.lockForConfiguration()
                device.torchMode = .off
                device.flashMode = .off
                device.unlockForConfiguration()
            }
        }catch{
            //DISABEL FLASH BUTTON HERE IF ERROR
        }
    }
    
    func captured(image: UIImage) {
        self.previewImage.image = image
        if let cgImage = image.cgImage, !settingImage {
            settingImage = true
            DispatchQueue.global(qos: .userInteractive).async {[unowned self] in
                self.currentImage = CIImage(cgImage: cgImage)
            }
        }
    }
    
    func cropImage(image: UIImage, toRect: CGRect) -> UIImage? {
        // Cropping is available trhough CGGraphics
        let cgImage :CGImage! = image.cgImage
        let croppedCGImage: CGImage! = cgImage.cropping(to: toRect)
        
        return UIImage(cgImage: croppedCGImage)
    }
    
    func addEmoji(id: String) -> String {
        return ""
    }
    
    //core ml model being embedded
    func detectScene(image: CIImage) {
        guard let model = try? VNCoreMLModel(for: food().model) else {
            fatalError()
        }
        // Create a Vision request with completion handler
        let request = VNCoreMLRequest(model: model) { [unowned self] request, error in
            guard let results = request.results as? [VNClassificationObservation],
                let _ = results.first else {
                    self.settingImage = false
                    return
            }
            
            //create thread
            DispatchQueue.main.async { [unowned self] in
                if let first = results.first {
                    if Int(first.confidence * 100) > 1 {
                        self.iSee.text = "Detecting: \(first.identifier) \(self.addEmoji(id: first.identifier))"
                        self.settingImage = false
                    }
                }
                //        results.forEach({ (result) in
                //          if Int(result.confidence * 100) > 1 {
                //            self.settingImage = false
                //            print("\(Int(result.confidence * 100))% it's \(result.identifier) ")
                //          }
                //        })
                // print("********************************")
                
            }
        }
        let handler = VNImageRequestHandler(ciImage: image)
        DispatchQueue.global(qos: .userInteractive).async {
            do {
                try handler.perform([request])
            } catch {
                print(error)
            }
        }
    }
}






