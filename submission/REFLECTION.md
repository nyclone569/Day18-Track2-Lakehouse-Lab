# REFLECTION — Day 18 Lakehouse Lab

Anti-pattern mà team mình thấy dễ vướng nhất là **small-file problem** — ghi quá nhiều file nhỏ vào Bronze layer.

Lý do: thường toàn làm prototype kiểu "chạy được là
được". Mỗi lần có batch mới về thì cứ `append` thẳng vào Delta table,
không ai nghĩ tới chuyện một ngày nào đó table sẽ có vài nghìn file
vài chục KB. Lúc làm NB2 mới thấy rõ: chỉ 200 lần append × 5K
rows thôi mà query đã chậm gấp 7 lần so với sau khi `OPTIMIZE +
Z-ORDER` (80ms → 11ms, 200 files → 55 files). Nhân lên với streaming ingestion thật chạy 24/7 thì con số sẽ kinh khủng hơn nhiều.

Cái nguy hiểm là nó không "fail" — pipeline vẫn chạy, dashboard vẫn
load, chỉ là chậm dần đều. Đến lúc ai đó than phiền thì table đã có
hàng chục nghìn file, fix lại mất công gấp bội.

Bài học rút ra: ngay từ đầu phải có lịch `OPTIMIZE` định kỳ (kiểu
nightly job) và Z-order theo cột hay filter nhất, đừng đợi đến khi
chậm mới đi sửa. Schema enforcement và partition layout cũng phải
nghĩ trước, không phải để sau.
