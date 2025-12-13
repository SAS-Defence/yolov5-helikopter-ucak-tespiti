# Helikopter ve Uçak Tespiti (YOLOv5 + SORT)

Bu repo, YOLOv5 kullanılarak eğitilmiş bir **helikopter ve uçak tespit modeli** içerir.  
Model hem **fotoğraflar üzerinde tespit**, hem de **videolarda SORT algoritması ile ID’li takip** yapabilmektedir.

---
## Kurulum

Bu repo yalnızca **eğitilmiş model ağırlığı ve takip scriptlerini** içerir.  
Modeli çalıştırmak için önce YOLOv5 kodlarının indirilmesi gerekir.

### YOLOv5’i indir
```bash
git clone https://github.com/ultralytics/yolov5
cd yolov5

## Repo İçeriği

- `weights/best.pt`  
   Eğitilmiş ve iyileştirilmiş YOLOv5 model ağırlıkları

- `results/`  
  Eğitim sonrası alınan performans grafikleri  
  - mAP ve loss grafikleri  
  - PR Curve  
  - Confusion Matrix  

- `tracking/`  
 Video takibi için SORT entegrasyonu  
  - `sort.py`  
  - `track_sort.py`

## Model ve Eğitim Bilgileri

- **Framework:** YOLOv5 (PyTorch)
- **Sınıflar:**
  - `0`  helicopter  
  - `1`  airplane
- **Görüntü boyutu:** 832 × 832
- **Batch size:** 16
- **GPU:** NVIDIA RTX 4070 Ti SUPER

## Eğitim Süreci (ÖNEMLİ)

Bu model **tek seferde sıfırdan eğitilmemiştir**, kademeli olarak geliştirilmiştir.

### İlk Eğitim

- İlk aşamada model, mevcut bir YOLOv5 ağırlığı üzerinden eğitilmiştir.
- Bu aşamada **60 epoch** eğitim yapılmıştır.
- Temel helikopter ve uçak ayrımı öğrenilmiştir.

### Model İyileştirme (Fine-Tuning)
- Daha sonra modele **ek 16.698 tane helikopter ve uçak görüntüleri** eklenmiştir.
- Özellikle:
  - Yan yana objeler
  - Farklı açılar
  - Daha zor sahneler
- Bu yeni verilerle model **tekrar eğitilmiştir (fine-tuning)**.

- **Fine-tuning epoch sayısı:** **30 epoch**

Yani model toplamda:
- **≈ 90 epoch eşdeğeri** bir öğrenme sürecinden geçmiştir  
- Son aşamada amaç, **genelleme kabiliyetini artırmak** ve pratik kullanımda daha stabil sonuçlar almaktır.

## Data Augmentation
Eğitim sırasında YOLOv5’in varsayılan veri artırma (augmentation) yöntemleri kullanılmıştır:

- Mosaic augmentation
- Random scale ve translate
- Random perspective
- HSV (renk) değişimleri
- Horizontal flip (uygun durumlarda)
Bu sayede model:
- Farklı ışık koşullarına
- Farklı ölçek ve açılara
daha dayanıklı hale getirilmiştir.

## Nihai Model Performansı (best.pt)

- **Precision:** %86.7  
- **Recall:** %89.6  
- **mAP@0.5:** %90.5  
- **mAP@0.5:0.95:** %52.9  

Sınıf bazında:
- **Helikopter:** mAP@0.5 = %88.2  
- **Uçak:** mAP@0.5 = %92.7  

## Çalıştırma Komutları : Aşağıdaki çok satırlı komutlar **Windows CMD** içindir. PowerShell kullanıyorsanız komutları **tek satır** halinde çalıştırın. 

-FOTOĞRAF İÇİN : 
py -3.11 detect.py ^
  --weights runs/train/heli_plane_moredata_ft/weights/best.pt ^
  --source "Fotoğrafın konumu" ^
  --img 832 ^
  --conf-thres 0.30 ^
  --iou-thres 0.25 ^
  --max-det 300 ^
  --device 0 ^
  --save-txt

-VİDEO İÇİN : 
py -3.11 track_sort.py ^
  --weights runs/train/heli_plane_moredata_ft/weights/best.pt ^
  --source "Videonun konumu" ^
  --img 832 ^
  --conf-thres 0.30 ^
  --iou-thres 0.25 ^
  --device 0

> Not: Fine-tuning sonrası metrikler, daha zor ve gerçekçi sahneler içeren validation seti üzerinden alınmıştır.

   **ABUSAS TEAM**