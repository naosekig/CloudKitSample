# CloudKitSample
Sample code for CloudKit

## INSERT
```
    private func insertData(code:String,name:String,costRate:Double,salesPrice:Int,image:UIImage!){
        let ckDatabase = CKContainer.default().privateCloudDatabase
        
        let ckRecord = CKRecord(recordType: "GoodsMaster")
        ckRecord["code"] = code
        ckRecord["name"] = name
        ckRecord["costRate"] = costRate
        ckRecord["salesPrice"] = salesPrice
        
        if (image != nil){
            let imageData = image.jpegData(compressionQuality: 1.0)
        
            var imageOrientation:Int = 0
            if (image.imageOrientation == UIImage.Orientation.down){
                imageOrientation = 2
            }else{
                imageOrientation = 1
            }
            
            ckRecord["image"] = imageData
            ckRecord["imageOrientation"] = imageOrientation
        }
        
        ckDatabase.save(ckRecord, completionHandler: { (ckRecords, error) in
            if error != nil {
                print("\(String(describing: error?.localizedDescription))")
            }
        })
    }
```

## SELECT
```
    private var goodsMasters:[GoodsMaster] = [GoodsMaster]()

    struct GoodsMaster {
        var code:String
        var name:String
        var costPrice:Double
        var salesPrice:Int
        var image:UIImage!
    }

    private func searchData(minSalesPrice:Int,maxSalesPrice:Int){
        let ckDatabase = CKContainer.default().privateCloudDatabase
        let ckQuery = CKQuery(recordType: "GoodsMaster", predicate: NSPredicate(format: "salesPrice >= %d and salesPrice <= %d", argumentArray: [minSalesPrice,maxSalesPrice]))
        
        ckQuery.sortDescriptors = [NSSortDescriptor(key: "salesPrice", ascending: false),NSSortDescriptor(key: "costRate", ascending: true)]
        
        ckDatabase.perform(ckQuery, inZoneWith: nil, completionHandler: { (ckRecords, error) in
            if error != nil {
                print("\(String(describing: error?.localizedDescription))")
            }else{
                self.goodsMasters.removeAll()
                for ckRecord in ckRecords!{
                    var image:UIImage! = nil
                    if (ckRecord["image"] != nil){
                        image = UIImage(data: ckRecord["image"]!)
                        let imageOrientation:Int = ckRecord["imageOrientation"]!
                        if (imageOrientation == 2) {
                            image = UIImage(cgImage: image!.cgImage!, scale: image!.scale, orientation: UIImage.Orientation.down)
                        }
                    }
                    let goodsMaster = GoodsMaster(code: ckRecord["code"]!, name: ckRecord["name"]!, costPrice: ckRecord["costRate"]!, salesPrice: ckRecord["salesPrice"]!,image:image)
                    self.goodsMasters.append(goodsMaster)
                }
                DispatchQueue.main.async {
                    self.tableView.reloadData()
                }
            }
        })
    }
```

## UPDATE
```
    private func updateData(whereCode:String,updateName:String,updateCostRate:Double,updateSalesPrice:Int,updateImage:UIImage!){
        let ckDatabase = CKContainer.default().privateCloudDatabase
        
        //1.更新対象のレコードを検索する
        let ckQuery = CKQuery(recordType: "GoodsMaster", predicate: NSPredicate(format: "code == %@", argumentArray: [whereCode]))
        ckDatabase.perform(ckQuery, inZoneWith: nil, completionHandler: { (ckRecords, error) in
            if error != nil {
                print("\(String(describing: error?.localizedDescription))")
            }else{
                //2.検索したレコードの値をUPDATEする
                for ckRecord in ckRecords!{
                    ckRecord["name"] = updateName
                    ckRecord["costRate"] = updateCostRate
                    ckRecord["salesPrice"] = updateSalesPrice
                    if (updateImage != nil){
                        //UIImageをNSDataに変換
                        let imageData = updateImage.jpegData(compressionQuality: 1.0)
                        
                        //UIImageの方向を確認
                        var imageOrientation:Int = 0
                        if (updateImage.imageOrientation == UIImage.Orientation.down){
                            imageOrientation = 2
                        }else{
                            imageOrientation = 1
                        }
                        
                        ckRecord["image"] = imageData
                        ckRecord["imageOrientation"] = imageOrientation
                    }
                    
                    ckDatabase.save(ckRecord, completionHandler: { (ckRecord, error) in
                        if error != nil {
                            print("\(String(describing: error?.localizedDescription))")
                        }
                    })
                }
            }
        })
    }
```

## DELETE
```
    private func deleteData(whereCode:String){
        let ckDatabase = CKContainer.default().privateCloudDatabase
        
        //1.削除対象のレコードを検索する
        let ckQuery = CKQuery(recordType: "GoodsMaster", predicate: NSPredicate(format: "code == %@", argumentArray: [whereCode]))
        ckDatabase.perform(ckQuery, inZoneWith: nil, completionHandler: { (ckRecords, error) in
            if error != nil {
                print("\(String(describing: error?.localizedDescription))")
            }else{
                //2.検索したレコードを削除する
                for ckRecord in ckRecords!{
                    ckDatabase.delete(withRecordID: ckRecord.recordID, completionHandler: { (recordId, error) in
                        if error != nil {
                            print("\(String(describing: error?.localizedDescription))")
                        }
                    })
                }
            }
        })
    }
```
