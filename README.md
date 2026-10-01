# krsmApp-ver1.01
自分で現在のコードの問題点を認識するために変更点を再整理するためのもの。
# 整理項目
初めに開くwindowを```Selectview```にすることで初めに管理者側のログインとお客様のログインで分ける。
```
import SwiftUI
import SwiftData
@main
struct KrsmaApp:App{
    var body:some Scene{
        WindowGroup{
            SelectView()
        }
        .modelContainer(for:Reservation.self)
    }
}
```
データ設計として永続していきたい運営者側の設定とお客様の予約を作る必要があり運営側の設定を追加していきたい。
```
//データ設計
@Model
class Reservation{
    var dateTime:Date
    var name:String
    var service:String
    init(dateTime:Date,name:String,cut:String){
        self.dateTime=dateTime
        self.name=name
        self.service=service
    }
}
```
```@Environment(\.dismiss) private var dismiss```の部分は元画面を消して次のページを新しく開くようにしたがページ変容をする際にこれが必要なのかは理解不足。
```
//利用者選択画面
struct SelectView:View{
    @Environment(\.dismiss) private var dismiss
    var body:some View{
        NavigationStack{
            Text("ヘアーサロンキリシマ")
                .font(.title)
            NavigationLink("お客様として利用"){
                ContentView()
ここを変更                dismiss()
            }
            .padding()
            NavigationLink("店舗管理者としてログイン"){
                MusterPassView()
ここを変更                dismiss()
            }
            .padding()
        }
    }
}
```
こちらのログイン画面も同様に```@Environment(\.dismiss) private var dismiss```理解不足。

コード内で```MusterPassWord```を変えればパスワードを変更できる。

```MusterView(records:$records)```で予約データを```records```として送る。

※```records```の内容を再度理解する。
```
//管理者ログイン画面
struct MusterPassView:View{
    @Environment(\.dismiss) private var dismiss
    @State private var MusterPassWord="Ykousuke0715"
    @State private var pass=""
    var body:some View{
        Text("店舗管理者ログイン")
        TextField("パスワードを入力",text:$pass)
        Button("ログイン"){
            if pass==MusterPassWord{
                MusterView(records:$records)
                dismiss()
            }
        }
    }
}
```
```@Query,let```などの定義や使い方の理解が低い。

```startTime```の部分で値を入力することで始業時間を設定。

```holiday```で休日を入力。

```
DatePicker(
                "予約者日付検索",
                selection:$reserti,
                in:Date()...,
                displayedComponents:[.date]
            )
            .datePickerStyle(.graphical)
```
```DatePicker```で予約者の有無を確認したい日を選択。
```
Button("検索"){
                showResult=true
            }
            if showResult == ture{
```

```
//管理者編集画面
struct MusterView:View{
    let records:[Reservation]
    @Query private var holiday:Int=0
    @Query private var startTime:Int=0
    @State private var reserti=Date()
    @State private var showResult=false
    var body:some View{
        VStack{
            TextField("休日設定",text:$holiday)
                .textFieldStyle(.roundedBorder)
            TextField("始業時間",text:$startTime)
                .textFieldStyle(.roundedBorder)
            DatePicker(
                "予約者日付検索",
                selection:$reserti,
                in:Date()...,
                displayedComponents:[.date]
            )
            .datePickerStyle(.graphical)
            Button("検索"){
                showResult=true
            }
            if showResult == ture{      
                Text("検索結果")
                List{
                    ForEach(records){reservation in
                        if reservation.dateTime == reserti{
                            Text("予約時間:\(reservation.dateTime) 名前:\(reservation.name) カット:\(reservation.cut)")
                        }
                    }
                } 
            } 
        }
    }
}

            

//利用者予約日時画面
struct ContentView:View{
    @Query private var records:[Reservation]
    @State private var selectedDate=Date()
    @State private var selectedHourMin=Date()
    var isBooked:Bool{
        records.contains{reservation in
            Calendar.current.isDate(
                reservation.dateTime,
                equalTo:selectedDate,
                toGranularity:.minute
            )
        }
    }
    var body:some View{
        NavigationStack{
            //日時選択
            DatePicker(
                "日付を選択",
                selection:$selectedDate,
                in:Date()...,
                displayedComponents:[.date]
            )
            .datePickerStyle(.graphical)
            NavigationLink("時間選択"){
                DetailhmView(selectedDate:selectedDate)
            }
=================ここはstruct DetailhmViewで実行させる===========================

            //予約状況
            if isBooked{
                Text("この日時は予約済みです。")
                    .foregroundStyle(.red)
            }else{
                Text("この日時は予約できます。")
                    .foregroundStyle(.green)
            }
==================================================================================
            NavigationLink(){
                CutDetailSelectView(selectedDate:$selectedDate)
            }label: {
                    Text("この日時で予約")
                }
                .disabled(isBooked)
                Divider()
        }
        .navigationTitle("予約")
        .padding()
    }
}

struct DetailHourMinView:View{
    let selectedDate:Date
    @Environment(\.dismiss) private var dismiss
    @State private var resevationtime=Date()
    let holiday:Date
    let startTime:Int
    var endTime:Date{
        let calendar=Calendar.current
        let weekday=calendar.component(
            .weekday,
            from:selectedDate
        )
        var hour=18
        if weekday == 1 || weekday == 7 {
            hour=17
        }
        return calendar.date(
            bySettingHour:hour,
            minute:0,
            second:0,
            of:selectedDate
        )!
    }
    let weekday=Calendar.current.component(
        .weekday,
        from:selectedDate
        )
    let holidays=Calendar.current.componet(
        .weekday,
        from:holiday
        )
    var body:some View{
        if weekday == holidays || weekday == 2{
            Text("休日")
            Button("終了"){
                dismiss()
            }
        else{
            DatePicker(
                "時間を選択",
                selection:$reservationtime,
                in:startTime...endTime,
                displayedComponents:[.hourAndMinute]
            )
        }




struct CutDetailSelectView: View {
    // 前の画面から受け取った日時
    let selectedDate: Date
    @Environment(\.dismiss) private var dismiss
    @Environment(\.modelContext) private var modelContext
    @State private var name = ""
    @State private var selectedService = ""
    let services: [String] = [
        "調髪：4400円",
    "調髪顔剃り無し：4100円",
    "調髪のみ：3600円",
    "アイロン：4900円",
    "SPなし：4100円",
    "顔剃りSP：3900円",
    "SPセット：2600円",
    "顔剃り：2600円",
    "セット：1200円",
    "丸刈り：3300円",
    "女性顔剃り：3300円",
    "高校生調髪：3800円",
    "高校生カット：3600円",
    "スキンフェイド：6400円",
    "中学生調髪：3300円",
    "中学生丸刈り：2100円",
    "小学生調髪：2700円",
    "小学生丸刈り：1800円",
    "乳児：3100円",
    "パーマ：9400円〜",
    "パーマ染：15000円",
    "アイパー：8100円〜",
    "Sパーマ：13000円",
    "Sパーマ前：7900円",
    "Sパーマ染：15000円",
    "Cut 白髪染：6600円",
    "Cut カラー：7600円",
    "白髪染めのみ：4200円"
    ]
    var body: some View {
        VStack(spacing: 20) {
            Text("情報入力")
                .font(.title)
                .bold()
            // 選んだ日時を表示
            Text(
                selectedDate,
                format: .dateTime
                    .year()
                    .month()
                    .day()
                    .hour()
                    .minute()
            )
            TextField("名前", text: $name)
                .textFieldStyle(.roundedBorder)
            Text("メニューを選択")
                .font(.headline)
            List {
                ForEach(services, id: \.self) { service in
                    Button {
                        selectedService = service
                    } label: {
                        HStack {
                            Text(service)
                            Spacer()
                            if selectedService == service {
                                Image(systemName: "checkmark")
                            }
                        }
                    }
                }
            }
            if selectedService != "" {
                Text("選択中：\(selectedService)")
            }
            Button("予約完了") {
                let record = Reservation(
                    dateTime: selectedDate,
                    name: name,
                    service: selectedService
                )
                modelContext.insert(record)
                dismiss()
            }
            .disabled(
                name.isEmpty ||
                selectedService.isEmpty
            )
        }
        .padding()
        .navigationTitle("予約内容")
    }
}
```
