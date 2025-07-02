C#:
1.ref, out, in
	-Đều truyền vào hàm kiểu tham chiếu
	-ref: truyền vào khi muốn thay đổi giá trị, cần gán trước khi truyền
	-out: truyền vào khi muốn lấy giá trị ra, gấn giá trị trước khi kết thúc hàm.
	-in: truyền vào để chỉ đọc, ko thể thay đổi giá trị áp dụng cho truyền vào struct lớn
	vì struct là value type và truyền vào hàm sẽ bị clone ra nên dùng in sẽ ngăn ko cho clone ra, mà sẽ thao tác trước tiếp, đồng đời ko thể thay đổi struct.
2. memcached và resdis cache
	2.1 Redis:
	-CSDL key-value dạng in-memory:
		+là data structure server, hỗ trợ nhiều kiểu dữ liệu string, list, set, hash, sorted set, stream
		+thích hợp làm mesagequeue và ranking system-xếp hạng người dùng; pub/sub- hệ thông sự kiện
	-Hỗ trợ dữ liệu phức tạp
	-Persist(lưu trữ) xuống disk(RDB,AOF): theo định kì
	+RDB: theo snapshoot(chụp lại bố nhớ tại 1 thời điểm) định kỳ toàn bộ dữ liệu -> ghi ra file .rdb
			Có thể tạo trigger: bao lâu 1 lần, đủ 1000key change thì chụp lại, hoặc thủ công dùng lệnh
		-> ko realtime, có thể mất giữ liệu giữa 2 lần snapshoot(crash, mất điện)-> ko đảm bảo khi hệ thống yêu cầu ko mất dữ liệu		
	+AOF: ghi lại lệnh thao tác (vào file .aof)để phục hồi: khi redis restart thì sẽ replay lại từng lênh để phục hội lại
		-> ít mất dữ liệu, file lưu sẽ lớn dần và phải replay nhiều-> chậm khi khởi động.
	có thể kết hợp 2 cách để đảm bảo dữ liệu hơn.
	=> có thể dùng như DB dạng in-memory

	-Hỗ trợ clustering:Redis Cluster để scale ngang:
		+Phân tán dữ liệu, nhiều máy chủ redis liên kết với nhau để phan tán dữ liệu(các node)
	-Giao thức
	-Hỗ trợ TTL(time to live) theo key: cài đạt key này sống trong bao lâu thì expire
		+redis sẽ tự động dọn các key khi hết hạn-> ko cần làm thủ công.
	-Cơ chế LRU / Eviction: cơ chế xóa bớt key khi hết bộ nhớ, việc xóa này theo tiêu chí cài đặt, vd: ttl sống ngắn thì xóa trước
	-Bảo mật: Auth: bảo mật, bắt client verify, TLS: mã hóa dữ liệu, ACL: quản lý truy cập-phân quyền khi nhiều app dùng chung
	, Monitor command:xem lệnh redis đang chạy theo realtime-> debug, nhưng tốn tài nguyên

	* Cache kiểu sort trond redis: redis sẽ tự động sắp xếp dữ liệu mà ko cần server xử lý,
	server chỉ cần báo là sẽ cache dữ liệu này trong redis với kiểu sort, khi đó redis sẽ tự keep dữ liệu kiểu sort mà server ko cần quan tam tới xứ lý sort
	-> server chỉ cần vào đọc ra là đã có dữ liệu sort rồi.
	* Dùng kiểu list trong queue để làm messagequeue

	2.2 memcached:
	-Cached key-value đơn giản: pure memory cached, lưu ket-value dạng string: key luôn là string, value là string, hoặc là object nếu được serialize.(max 1MB sau serialize)
		+Dữ liệu mất khi restart, full RAM
		
	-chỉ string 
	-ko persist, thuần volatile:nghĩa là chỉ sống trong RAM, dữ liệu mất khi RAM mất điện
	-Scale ngang thủ công: nếu có  nhiều note thì pghair tự phân phối key sang các node tương ứng khi có thay đổi.
	- Giao thức:
	-Có TTL: time to live cho từng KEY
	=> Cần khi truy xuất nhanh, lưu object nhỏ và có TTL, khi ko cần replication or persist.
	=>Dùng cho Session, page fragment cache, result query Db
	
3. struct vs class
	3.1 struct:
	-kiểu giá trị(value type) - >Dữ liệu lưu trên stack-> truy cập nhanh hơn do lấy trực tiếp giá trị từ stack.
	-Truyền vào hàm: copy nội dùng
	-Ko thể kế thừa- dùng cho cấu trúc nhẹ
	-sẽ  bị boxing khi chuyenr sang object.interface
	
	3.2 class:
	-Kiểu tham chiếu(ref type) - > Dữ liệu lưu trên heap- truy cập chậm hơn do phải chiếu từ stack sang heap mới lấy dk giá trị thực
	-Truyền vào hàm là truyền địa chỉ trên stack, thao tác sẽ ảnh hưởng tới giá trị thực trên heap
	-Có kế thừa, dùng cho đối tượng có hành vi phức tạp
	-Ko bị boxing do nó sãn là kiểu ref đang trên heap rồi.

4. Boxing/Unboxing và tác động hiệu suất
-Boxing là quá trình đóng gọi 1 value type ở stack thành ref type để lưu ở heap
-Khi nào xảy ra:	
	+gán biến valuetype vào biến object, interface, 
	+truyền value type vào hàm nhận tham số là object/interface
-Unboxing: lấy 1 biến tù heap về stack
-Tác động tới hiệu suất: boxing rất tốn kém nếu xảy ra n hiều lần: vì quá trình cần tạo đối tượng mới, cấp phát trên heap, trình dọn rác GC làm việc
+unbox phải kiểm tra,tốn cpu, lỗi có thể xảy ra nếu ko kiểm tra tốt

-Tránh boxing như nào:
+Dùng List<> thay vì dùng arraylist
+cẩn thận khi dùng với struct vì dễ dính boxing-
+ nên Dùng Span<T>, ref struct: sẽ cấp phát bộ nhớ trên stack-> ko bị boxing. hay còn dùng đẻ ép value type luôn nằm trên stack.
-Dùng generic để tránh boxing như nào
+Dùng generic để khi trình biên nó sẽ dùng luôn kiểu dữ liệu T đó chứ ko chuyển sang object nữa(ko boxing nữa)
Có tác dụng khi T là các kiểu dữ liệu vlaue type sẽ ko bị chuyển sang kiểu object khi thao tác.
🧱 Câu thay thế:
	Cũ				Mới
	ArrayList	✅ List<T>
	Hashtable	✅ Dictionary<TKey, TValue>
	Queue		✅ Queue<T>
	Stack		✅ Stack<T>
5. Sự khác biệt giữa Task, Thread, async/await và ThreadPool
-Thread: Luồng do hệ điều hành cấp phát, quản lý manual: manual là dễ lỗi
+dùng khi tác vụ nền nặng, chạy dài cần kiểm soát.: xử lý ảnh video , file lớn,
+toàn quyền điều khiển(ưu tiên chạy, dừng, trạng thái)
+Tốn tài nguyên(1thread ~ 1MB stack)-> tạo nhiều cạn RAM
+tạo mới tốn thời gian, ko sử dụng lại được

-Threadpool: nền tảng tối ưu hóa thread.
+Pool các luồng tái sử dụng: bể chứa các luồng đã tạo sẵn, khi sử dụng ko cần mất time để tạo nữa
+>net tạo ra pool và cấp phát khi cần, sau khi thread chạy xong sẽ ko bị hủy mà quay lại trong pool để chờ lần tiếp sử dụng
+quản lý tự động
+dùng khi nhiều tác vụ nhỏ nhanh
-Task: mô hình lập trình song song hiện đại, trừu tượng trên thread và threadpoool(vì nó chỉ là công việc, thread nào chạy ko quan tâm)
+đại diện cho 1 công việc đang chạy, sẽ chạy, đã hoàn thành, lỗi hoặc hủy
+là 1 warpper cho 1 đơn vị công việc, công việc đó thực thi bởi threadpoool(khi dùng task.Run) hoặc luồng khác tùy khời tạo
+Dùng trong có đồng bộ, bất đồng bộ
+ dễ dungfm dùng với asyn/await
+Vẫn dùng threadpoool=> ko hợp với công việc dài, rất dài, blocking
-Async/await:Viết code bất đồng bộ dễ đọc như cứ đồng bộ.
+Dựa vào task, task<T> để thể hiện công việc thực hiện
+trành block thread, tối ưu I/O-bound task(HTTP,DB,File..)
+await ko tạo ra luồng mới, ko block luồng hiện tại, mà kệ cho tác vụ A thực hiện đồng thời giải phòng thread,
Giải phóng thread để thread có thể đi thực hiện các tác vụ X khác.
khi nào tác vụ A xong thì thread(có thể là thread cũ) tiếp tục thực hiện ở sau wait.
+.Reasult(). .wait có thể gây deadlock vì thread nó sẽ chở ở đây đến khi xong tác vụ mới chạy tiếp
Mẫu chuẩn asyn/await
public async Task<ActionResult> GetData()
{
    try
    {
        var data = await _service.LoadAsync();
        return Ok(data);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Lỗi load data");
        return StatusCode(500);
    }
}

6.Garbage Collection nâng cao: Gen 0/1/2, Finalizer, IDisposable
-GC:
+Hệ thống tự động thu gom rác trong.Net, giải phóng vùng nhớ ko còn dùng tới
+ko cần free thủ công hoặc delete như C++
+tránh lối leak memeory
+cơ chế: khi ko còn bất kì tham chiếu nào trở tói 1 object-> nó được coi là rác-> dọn
+việc dọn là theo cơ chế để tránh lạm dụng, đơ lag chậm trường trình: khi app chiếm nhiều ram, quá nhiều biến Gen0, ko đủ bộ nhớ cấp thêm cho biến
-GC gen 0/1/2:
+Gen0:Object mới tạo, sống ngắn
+Gen1:Object sống qua lần quét 1
+Gen 2: object sống dại, sống qua lần quét thứ 2,3,4...
+Phân chi ra như này để dễ quét, vì thường sẽ quét gen thấp nhiều hơn vì thời gian sống ngoắn
+thu gong riêng từng gen sẽ nhanh hơn
-Finalizer:
+Là phương thức đặc beiets dk .net gọi ngay trước khi đối tượng bị GC thu gom.
+khi object có Fi thì GC sẽ ko thu gom ngay mà phải đợi 1 thread finalizer thread chạy xong thì GC mới chạy dk
=>chậm quá trình giải phóng bộ nhớ-> nên tránh
-Object có finalizer sẽ sống lâu hơn bình thường
>Finalizer sử dụng khi làm việc với tại nguyên unmanaged
-C# ko nên dùng
-IDisposable:Giao diện interface dùng giải phóng tài nguyên đúng lúc thường là trước khi GC thu gom khi làm việc với file, network, steam DB, unmanaged memeory.
+Giúp giải phóng tài nguyên sóm hơn và rõ ràng hơn. tránh rò rỉ: file bị khóa, soket ko đóng, kết nối DB ko đóng.
vì ngay sau khi dùng xong thì GC chưa dọn ngay mà phải đợi trigger mới thực hiện. trong khoảng thời gian đó thì những luồng khác ko thể truy cập được.
+Dùng khi nào:
file: steam, streamreader
Kết nối db,
socket,timer, thread

7. Stack và heap
-Tính chất của 2 vùng nhớ này: cấu trúc tổ chức dữ liệu, tốc độ
+Stack: Last in first out, bộ nhớ nhỏ ~1MB/thread, cấp phát nhanh, lưu trũ value type, biến tạm, tham số, quản lý bởi complier & CPU, khi kết húc scope hoặc method thì xóa
+Heap: Dạng phân tán, ko có thứ tự, cấp phát chậm hơn, lưu biến reference type(object, array, class),quản lý bởi GC của .Net, dọn dẹp khi không còn tham chiếu.
-Biến nào trên stack và heap: 
+tất cả các biến value type năm trên stack:int, float, double, decimal, bool, char, byte, sbyte, short, ushort, long, ulong, DateTime, enum, struct, Guid
+các bến ref nằm trên heap:class, interface, delegate, string, object, array, dynamic
+Note: value type nằm tren heap khi nó là 1 field của một object, boxing sang kiểu object.
------------------------------------------------------------------------------------------
1. High performance .NET/Memory-optimized programming
2.Advanced multithreading & concurrency
3.Dependency injection (DI)& Ioc container
4.Advanced garbage collection tuning
5.Roslyn & complie-time programming
6. Networking &protocol level handling
7. .net internal & runtinme
8. Real world system design với .NET
9. Performance optimization tools
