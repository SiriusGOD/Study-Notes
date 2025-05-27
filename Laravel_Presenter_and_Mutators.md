# Laravel 架構下 Presenter 與 Eloquent Accessor & Mutator 使用時機
## 前言
實作上遇到一個需求是店舖列表會顯示徵人的薪資，但需要顯示換算後的單位，這時原先是在Presenter做格式化的function，並且在迴圈做調用，最後發現把function移至Eloquent Accessor (Model Attribute)裡速度會更快，就有了此次的研究

## Eloquent Accessor & Mutator
#### 使用時機
資料模型層級的轉換邏輯，通常與資料欄位直接對應，會常用在資料儲存與讀取時做統一格式處理
#### 適合處理：
* 格式化邏輯只跟單一 Model 資料有關（如薪資、日期、名稱等單一欄位的格式化）。
* 每筆資料都需要顯示格式化結果（例如列表、卡片、表格等）。
* 格式化結果可以快取，不需每次都重新計算（Eloquent accessor 會自動快取）。
* 格式化邏輯簡單、重複性高，如金額加單位、日期格式化、狀態轉換等。
* Blade 只需簡單輸出，如：{{ $model->formatted_salary }} 或 {!! $model->format_wage_html !!}。
#### 優點：
* 效能佳（同一物件只計算一次）。
* Blade 簡潔、易維護。
* 資料與格式化邏輯集中於 Model，方便重用。
#### 缺點：
* 無法做太複雜的邏輯（例如關聯模型、多條件格式）
* 和 View 綁得太緊時會難維護
#### 範例：
1. Laravel 8 或更早版本
```
public function getNameAttribute($value)
{
    return strtoupper($value);
}
```
2. Laravel 9 後
```
protected function name(): Attribute
{
    return Attribute::make(
        get: fn ($value) => strtoupper($value),
    );
}
```
3. 進階用法：也可搭配 Mutator（寫入時處理）
```
protected function name(): Attribute
{
    return Attribute::make(
        get: fn ($value) => strtoupper($value), // 讀取時轉大寫
        set: fn ($value) => strtolower($value), // 寫入時轉小寫
    );
}
```
## Presenter（或 ViewModel）
#### 使用時機
視圖/顯示層的邏輯處理，當你需要為 Blade、API、或前端格式化顯示資料時，不要污染 Model，可以用 Presenter。
#### 適合處理：
* 需要組合多個 Model 或多欄位資料，進行複雜的格式化或業務邏輯。
* 格式化結果與當前畫面/情境有關（如根據登入者、語系、權限等動態變化）。
* 需要產生複雜 HTML、包含多層結構或條件判斷。
* 同一資料在不同畫面需要不同格式化方式。
* 希望將 view 專用的邏輯與 Model 資料分離，讓 Model 保持乾淨。
#### 優點：
* 複雜邏輯集中於 Presenter，Model 保持單純。
* 易於測試與重構。
* 適合多 Model/多層資料組合。
#### 缺點：
* 要額外建立 class
* 使用時需多呼叫 .presenter() 或手動注入
#### 範例：
```
@inject('shopSalaryPresenter', 'App\Presenters\User\ShopSalaryPresenter')

<div class="employment-and-salary">
    <div class="is-employmentIcon {{ $employmentType }}">{{ $employmentIcon }}</div>
    {!! $shopSalaryPresenter->displaySalaryAdjustWithTag($employmentShopDetailJunction->firstSalary) !!}<br>
</div>
```
## 實務建議
* 單一欄位、單一 Model 的格式化（如金額、日期、狀態）：用 Model accessor（class attribute）。
* 跨 Model、跨欄位、複雜 view 專用邏輯：用 Presenter。
* 大量資料列表頁：盡量用 accessor，避免 Presenter 造成效能瓶頸。
* 特殊情境或一次性 view 組合：可用 Presenter。


## 小結
此次遇到的問題就是在view的迴圈中，原先使用Presenter去計算每間店鋪的薪資，因為每一次回圈都要計算 + 額外的 function call + 物件注入導致效能上比單純的 accessor 慢。accessor 只會在物件本身上運算，效能較佳。

資料參考：[laravel Eloquent: Mutators & Casting](https://laravel.com/docs/9.x/eloquent-mutators)