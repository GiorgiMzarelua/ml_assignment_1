# ml_assignment_1

# სახლების ფასების პროგნოზირება — Kaggle Competition

## კონკურსის მიმოხილვა
Kaggle-ის კონკურსი "House Prices: Advanced Regression Techniques" მოიცავს სახლების გაყიდვის ფასების პროგნოზირებას. ვაფასებთ RMSE მეტრიკით ლოგარითმულ სკალაზე.

## ჩემი მიდგომა
პრობლემის გადასაჭრელად ჯერ გავწმინდე მონაცემები, შევქმენი ახალი feature-ები, სუსტი feature-ები მოვაცილე და შემდეგ გავტესტე სხვადასხვა ექსპერიმენტები.

## რეპოზიტორიის სტრუქტურა
- `model_experiment.ipynb` — მონაცემების დამუშავება, ფიჩერ ინჟინერინგი, ფიჩერ სელექცია და მოდელების ტრენინგი
- `model_inference.ipynb` — საუკეთესო მოდელის გამოყენება test set-ზე და submission-ის გენერაცია
- `README.md` — პროექტის დეტალური აღწერა

## Feature Engineering

### Area Features
შევქმენი `TotalSF` feature, რომელიც აერთიანებს სარდაფის, პირველი და მეორე სართულის ფართობს. ეს ფიჩერი უფრო ძლიერი predictor-ია ვიდრე ცალკეული სართულების ფართობები — კორელაცია SalePrice-თან 0.78.

### Age Features
გარდავქმენი აბსოლუტური წლების სვეტები ფარდობით ასაკად:
- `HouseAge` — სახლის ასაკი გაყიდვის მომენტში. კორელაცია -0.52, რაც ლოგიკურია — ძველი სახლი იაფია
- `YearsSinceRemodel` — რამდენი წელი გავიდა რემონტიდან. კორელაცია -0.51
- `IsRemodeled` — გაკეთდა თუ არა რემონტი. 

### Bathroom Features
4 ცალკეული სველი წერტილის სვეტი გავაერთიანე ერთ `TotalBathrooms` სვეტში. ნახევარი სველი წერტილი ითვლება 0.5-ად. კორელაცია SalePrice-თან 0.63.

### Binary Flags
შევქმენი yes/no ტიპის binary flags. ყველაზე ძლიერი `HasFireplace` — კორელაცია 0.47. `HasPool` ძალიან სუსტია (0.09) რადგან ამ dataset-ში მცირე რაოდენობის სახლს აქვს აუზი.

### Quality Interaction Features
`QualXSF` = `OverallQual` × `TotalSF` — ხარისხი გამრავლებული ფართობზე. ეს ყველაზე ძლიერი ფიჩერია — კორელაცია 0.856, უფრო მაღალი ვიდრე ცალკე OverallQual (0.79).

### Ordinal Encoding
ხარისხის სვეტები (ExterQual, KitchenQual და სხვა) აქვთ ბუნებრივი თანმიმდევრობა Po < Fa < TA < Gd < Ex. ამ სვეტებს დავუნიშნე 0-5 რიცხვები თანმიმდევრობის შესანარჩუნებლად. ყველაზე ძლიერია ExterQual (0.68) და KitchenQual (0.66). ExterCond თითქმის უსარგებლოა — კორელაცია მხოლოდ 0.019.

### One-Hot Encoding
დარჩენილი კატეგორიული სვეტები გარდავქმენი one-hot encoding-ით. train და test ერთად დავა-encoding-ე კონსისტენტური სვეტებისთვის.

## Cleaning მიდგომები
- სვეტები სადაც NaN ნიშნავს "feature არ არსებობს"  შევავსე "None" მნიშვნელობით
- კატეგორიული სვეტები (MasVnrType, Electrical) შევავსე მოდით(ანუ ყველაზე ხშირად გამოყენებადი მნიშვნელობით)
- რიცხვითი სვეტები (MasVnrArea, GarageYrBlt) შევავსე ნულით
- LotFrontage შევავსე Neighborhood-ის მედიანით(უფრო ზუსტია ვიდრე გლობალური მედიანა)
- 2 outlier ამოვიღე — სახლები, რომელთაც ჰქონდათ დიდი ფართობი და დაბალი ფასი. 

## Feature Selection
- ამოვიღე 88 feature კორელაციით < 0.05 
- ამოვიღე დუბლიკატი feature-ები (კორელაცია > 0.95): YearBuilt/HouseAge, GarageYrBlt/HasGarage და ა.შ.
- საბოლოოდ 147-მდე დავიყვანეთ სასარგებლო feature-ები

## Training

### Linear Regression
პირველი ბეისლაინი — 31691 RMSE. მაღალი შედეგი იყო, თუმცა Log ტრანსფორმაციის შემდეგ RMSE 0.1310-მდე დაეცა.

### Ridge Regression
L2 რეგულარიზაცია. საუკეთესო alpha=0.1, RMSE 0.1309 — თითქმის იგივე Linear Regression-ის შედეგი.

### Lasso Regression
L1 რეგულარიზაცია — ავტომატურად ანულებს სუსტ ფიჩერებს. საუკეთესო alpha=0.0001, RMSE 0.1303. Outlier-ების მოცილების შემდეგ 0.1144-მდე გაუმჯობესდა.

### ElasticNet
L1 და L2 კომბინაცია. საუკეთესო alpha=0.0003, l1_ratio=0.95, RMSE 0.1298. Outlier-ების მოცილების შემდეგ 0.1145.

### Decision Tree
Default პარამეტრებით Train RMSE 0.001 და CV RMSE 0.20 — კლასიკური overfitting. max_depth=5-მა კი გააუმჯობესა CV RMSE 0.1785-მდე. min_samples_leaf=10-ით კიდევ 0.1739-მდე. თუმცა მაინც Linear Regression-ზე მაინც ცუდია და უარესი შედეგი დადო ამ კონკრეტულ სიტუაციაში.

### Random Forest
100 ხისგან შემდგარი ანსამბლი — Train RMSE 0.0503, CV RMSE 0.1354. Decision Tree-ზე უკეთესი შედეგი, მაგრამ Lasso-ს კვლავ ჩამორჩება.


### საბოლოო მოდელის შერჩევა
Lasso alpha=0.0003 outlier-ების მოცილებით — CV RMSE 0.1144. ეს მოდელი ავირჩიეთ რადგან:
- ყველაზე დაბალი CV RMSE
- L1 რეგულარიზაცია ავტომატურად ასელექტირებს feature-ებს
- Dataset-ის წრფივი ბუნება linear model-ს ხელს უწყობს

## MLflow Tracking
ექსპერიმენტების ბმული: https://dagshub.com/GiorgiMzarelua/ml_assignment_1.mlflow

### ჩაწერილი მეტრიკები
- `rmse_cv_mean` — cross-validation RMSE საშუალო
- `rmse_cv_std` — cross-validation RMSE სტანდარტული გადახრა
- `train_rmse` — სატრენინგო RMSE (Decision Tree-სთვის overfitting-ის საჩვენებლად)
- `overfit_gap` — train და CV RMSE-ს სხვაობა

### საუკეთესო მოდელის შედეგები
- CV RMSE: 0.1144
- Kaggle Public Score: 0.141
- Model Registry: house_prices_best_model version 4
