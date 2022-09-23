# 電子發票資料Turnkey交換範例
2018-07-26


https://dotblogs.com.tw/ProgrammerFighting/2018/07/26/163310

電子發票-TurnKey用XML相關參考
2018-07-30
前陣子做一個案子，需要產生政府用的電子發票，然後利用TurnKey上傳，政府文檔內容分散太多

讓人搞不太清楚重點，所以寫一篇文章順便做整理記錄

當初在規劃結構的時候有稍微參考 Githut其中一位前輩的方法

各位也可參考看看

電子發票資料Turnkey交換範例

完全沒開發過跟電子發票有關的東西，一直不清楚他們得流程

當初稍微看一下，想說有提供API，以為只要能串接API就能搞定整個流程

最後才發現大錯特錯。

政府電子發票API說穿了只是一些雞肋的功能而已，提供查詢阿，或是對獎用的API

跟電子發票開立和上傳，完全沒關係。

(B2C)真正發票開立和上傳系統綁定在TurnKey軟體上，需要產生發票的XML文檔，之後上傳到他們的整合平台才算開立發票成功。

那時候因為專案時間急迫，和網路上沒現成的範例能使用，只找到一個有關的Githut

各位可以參考看看 電子發票資料Turnkey交換範例 ，我後面所提供的範例檔，裡面的 EInvoiceXSD 也是出至此處

該交換範例所使用的方式，是使用資料類別來產生XML，因為我也需要利用資料類別串接SQL資料，所以剛好適用

不過結構上不太夠用，加上本來就是範例，所以後來整個結構就重寫，讓結構更適合我的專案。

之後我所提供的範例檔，並不能直接執行成為專案，因為專案有跟SQL結構綁在一起，我只提供XML的當參考使用

所以如果要使用我的範例檔，請自行修改以符合自己的專案。

並且，該範例檔，結構上並不是很完美，也有很多只是為了通過政府的系統，而簡單寫，不太會再去用到

像是產生實體發票的PDF檔，即是簡單撰寫，因為我的專案中，全部走電子發票載具流程(新稱法，雲端發票)，根本不會印製出來

但是政府需要提供印製實體發票的範本，才能去申請字軌。

所以有些東西只是簡單寫，並沒有打算要再修改，有需要的人請自行強化自己所需的部分

我的專案結構分為

DataClass-發票用的檔案結構類別
EInvoiceDataB2B-給B2B使用的類別，並不是很完整，因為一開始寫B2C所以B2C結構比較完整。

EInvoiceDataB2C-給B2C使用的類別，因為專案最主要用B2C所以算整體上最完整的。

EInvoiceDataBranchTrack- 字軌用的結構

EInvoiceDataClass-共通用的結構及enum定義

EInvoiceDataPlatform-平台用的結構，也不是很完整，本來要寫成讓平台回傳的文檔

可以開啟轉回結構檔案顯示在程式中，但是後來專案沒用到，就沒去補充

EInvoiceHelper-電子發票XML文檔產生相關類別
EInvoiceQrHelper-電子發票實體列印相關的類別，利用itextsharp.dll 簡單做一個 PDF，有需要的請自行修改

XmlHelperAll-產生Xml用的共通性類別

XmlHelperB2B-空的，因為沒用到B2B所以就沒寫

XmlHelperB2C- For B2C用類別

XmlHelperForTesting- For Turnkey自行檢測所需xml檔的類別，但是寫完沒測試，因為第一次跑官方流程，所以一開始先自己直接產生發票

之後了解官方流程後才回來寫這個，所以就沒再測試了

XmlHelperPlatform- For 平台回傳或是其他的xml用類別

EInvoiceXSD
驗證用的XSD檔案，檔案出處請參考
https://github.com/RichieHsu/ENVITest/issues/1

OtherPackages- For 列印發票範本使用 相關DLL
itextsharp.dll- 以前舊專案Copy來的，好像才2的樣子，現在都已經 iText7了，不過7要收費，再NuGet可以直接下載iText5版本來用

QREncrypter.dll-電子發票官方的文檔 加解密工具 裡面提供的Dll，因為太多我也忘記是從哪個檔案裡面找到
的請自行在這裡研究吧
https://www.einvoice.nat.gov.tw/ein_upload/html/1428905476324.html

再說明一次，我提供的範例檔案並不是完整專案檔

就算開個空專案全部放進去，也不一定能執行(因為有些結構是SQL結構，我沒放進來)，可能會有些缺失

只是提供一些 for Turnkey 的程式參考範例而已，給一些開發人員啟發

不太適合全新手使用。使用上有問題歡迎留言提問

範例檔案下載
https://goo.gl/gwq24y

如果我之前文章說到，感覺沒貼一些程式碼，就是怪怪的

所以下面會解說一些內容，但並不全面

EInvoiceDataClass.cs
這裏面放的是 轉換成Xml用的一些基礎類別

其中 Master屬性，代表再XML為必要的，再產生XML時會先檢查

錯誤會直接中斷再程式裡面，並不會產生XML文件檔

[AttributeUsage(AttributeTargets.Property)]
public class MasterAttribute : Attribute
{
}
如果通過，再利用XSD檔案作格式跟SIZE的檢查。但是XML檔案已經產生

所以如果XSD檔案檢查沒過，就要自己再去把文件檔案移除掉

然後 enum 有給數字的，都代表要取數字出來用，而沒給數字直接流水編碼的代表要取文字使用

EInvoiceDataB2C.cs
namespace EInvoiceDataClass.B2C
發票類別都利用 namespace來做區分，所以程式要同時用到兩個(B2C和B2B)，請自行修改

在Invoice類別裡面，提供一個產生標準(Master欄位)發票類別，產生出來之後外部程式在自行修改細節

(產生出來的是For 我專案用的，所以內容請自行修改)

// 產生標準的Invoice 資料
public static Invoice GenerateStandardInvoiceData(int ProductItemQuantity = 1)
{
    Random rd = new Random();
    Invoice rtn = new Invoice();
    var main = rtn.Main;
    // 發票號碼外面補
    main.InvoiceDate = DateTime.Now.ToString("yyyyMMdd");
    main.InvoiceTime = DateTime.Now.ToString("HH:mm:ss");
    var seller = main.Seller;
    seller.Identifier = "公司統編";
    seller.Name = "公司名稱";
    //seller.Address = "公司地址";
    //seller.PersonInCharge = "負責人名稱";
    //seller.TelephoneNumber = "聯絡電話";
    //seller.FacsimileNumber = "傳真電話";
    //seller.EmailAddress = "信箱資料";
 
    var buyer = main.Buyer;
    buyer.Identifier = "0000000000";
    buyer.Name = rd.Next(0, 10000).ToString("D4");
    // 檢查碼不可寫      
    main.InvoiceType = InvoiceTypeEnum.GeneralTax;
    main.DonateMark = DonateMarkEnum.NotDonate;
    main.CarrierType = EInvoiceStaticData.CarrierTypeEnumType1;
    main.PrintMark = "N";
    main.RandomNumber = rd.Next(0, 10000).ToString("D4");
 
    // Details
    var productItem = new InvoiceProductItemDetails();
    productItem.Description = "產品敘述";
    productItem.Quantity = ProductItemQuantity;
    productItem.UnitPrice = 1000;  // 售價
    productItem.Amount = (productItem.Quantity * productItem.UnitPrice);
    rtn.Details.ProductItem.Add(productItem);
    productItem.SequenceNumber = rtn.Details.ProductItem.Count.ToString("D3");
 
    var amount = rtn.Amount;
    amount.SalesAmount = productItem.Amount;
    amount.FreeTaxSalesAmount = 0;
    amount.ZeroTaxSalesAmount = 0;
    amount.TaxType = TaxTypeEnum.Tax;
    amount.TaxRate = 0.05M;
    amount.TaxAmount = 0;
    amount.TotalAmount = Convert.ToInt64(amount.SalesAmount);
 
    return rtn;
}
資料類別大約就這些值得說一下的，另外因為我專案只需要用B2C

後來B2B只是順便補充進去，所以會感覺B2B寫得挺草率的。

EInvoiceQrHelper.cs
這是要產生印製發票的範本PDF，有需要的可以參考研究看看

另外iText的QR因為一開始查不到使用UTF-8的方式，所以轉成圖片都會不見

所以我專案是用Base64產生圖片，但是後來找到方法，只是已經通過驗證了，所以我就沒再去用了

利用下面這方式，可以產生UTF-8的圖片

// 如要使用 UTF-8的QR圖方法
var hints = new Dictionary<EncodeHintType, Object>() { { EncodeHintType.CHARACTER_SET, "UTF-8" } };
//byte[] bytes = Encoding.UTF8.GetBytes(temp.ToString());
//int byteLength = bytes.Length; // Max 133，超過圖片大小會不同
//var QrImageLeft = new BarcodeQRCode(temp.ToString(), 1, 1, hints).GetImage();
//QrImageLeft.ScaleToFit(new Rectangle(0, 0, 46, 46)); // 提供另一個方式去設定圖片大小
XmlHelperAll.cs
基礎共通性的類別

其中我把XSD驗證 註解不用，因為我格式都是固定死的，並不會變更，所以就不做檢查

有需要檢查的人使用前記得要先設定

// Xsd檔案放置的位置
public static string CheckXsdPath { get; set; }
最後是每個XML檔案都要設定的格式問題，這問題查了許久才找到解法

最主要是設定xsi要去做XSD驗證使用，xsi格式會跟標準的不同

// xsi 會消失
//rootXmlElement.SetAttribute("xmlns:xsi", "http://www.w3.org/2001/XMLSchema-instance");
//rootXmlElement.SetAttribute("xsi:schemaLocation", $"{CommonData.Xmlns} {CommonData.XsdName}");
 
// 自動產生的不是xsi
//rootXmlElement.SetAttribute("schemaLocation", "http://www.w3.org/2001/XMLSchema-instance", $"{CommonData.Xmlns} {CommonData.XsdName}"); 
 
// 理論上可運作，不過跟範例的Attribute順序不同
XmlAttribute attribute = doc.CreateAttribute("xsi", "schemaLocation", "http://www.w3.org/2001/XMLSchema-instance");
attribute.Value = $"{CommonData.Xmlns} {CommonData.XsdName}";
rootXmlElement.SetAttributeNode(attribute);
大致上就這些吧

剩下的就一些做工的部分而已，另外利用遞迴方式去抓類別的名稱跟數值來轉成XML檔。

這部分並沒有寫得很好看，不過運作起來以符合我專案內容了

所以可能會有其他BUG或是有更好優化的方式。歡迎留言討論

範例檔案下載
https://goo.gl/gwq24y
