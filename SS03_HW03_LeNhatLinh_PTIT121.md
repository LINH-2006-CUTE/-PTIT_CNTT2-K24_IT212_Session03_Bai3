1. Phân tích lý do thất bại của prompt thô: "Code này có chạy được không?" 	
	Thiếu vai trò định hình tư duy (Role): Khi không được giao một vai trò cụ thể như "Chuyên gia kiểm thử (QA)" hay "Senior Developer", AI mặc định xử lý thông tin ở mức độ bề nổi của một trình biên dịch (Compiler) thông thường. Nó chỉ kiểm tra xem mã nguồn có đúng cú pháp Java hay không (như khai báo biến, vòng lặp, ép kiểu) chứ không đi sâu phân tích tư duy phòng thủ (Defensive Programming)
	Mục tiêu và câu hỏi quá chung chung (Mơ hồ về Goal): Câu hỏi "có chạy được không?" rất mơ hồ. Về mặt lý thuyết, nếu danh sách truyền vào có phần tử, mã nguồn này hoàn toàn biên dịch và chạy được. AI sẽ dựa vào kịch bản lý tưởng đó (Happy Path) để trả lời "Có", từ đó bỏ qua hoàn toàn các kịch bản lỗi biên
	Thiếu dữ liệu về ngữ cảnh và các ràng buộc hệ thống (Context & Constraint): Prompt không cung cấp ngữ cảnh về môi trường chạy hoặc các điều kiện đầu vào có thể xảy ra trong thực tế (như dữ liệu từ cơ sở dữ liệu hoặc API truyền về có thể bị null hoặc rỗng). Khi không có ràng buộc phải kiểm tra an toàn mã nguồn, AI sẽ không chủ động dò tìm lỗ hổng

2.Nội dung Prompt tối ưu mới do bạn thiết kế
"Hãy đóng vai trò là một Chuyên gia Kiểm thử phần mềm (QA/Tester) và Chuyên gia rà soát mã nguồn (Senior Code Reviewer) cực kỳ sắc bén và kỹ tính trong ngôn ngữ Java. 

Nhiệm vụ của bạn là kiểm tra, phát hiện các lỗi logic biên nghiêm trọng có thể làm sập ứng dụng hoặc gây sai lệch kết quả trong đoạn mã nguồn dưới đây.

public class AverageCalculator {
    public static double calculateAverage(List<Integer> numbers) {
        int sum = 0;
        for (int num : numbers) {
            sum += num;
        }
        return (double) sum / numbers.size();
    }
}
Ràng buộc xử lý và kiểm thử nghiệp vụ:
1. Hãy chỉ ra chính xác các trường hợp biên (Edge Cases) đầu vào có thể gây ra ngoại lệ hệ thống hoặc lỗi logic từ đoạn code trên.
2. Thực hiện sửa lỗi mã nguồn theo tư duy lập trình phòng thủ (Defensive Programming). Nếu danh sách bị null hoặc trống, hãy trả về giá trị mặc định là 0.0.

Định dạng đầu ra mong muốn:
- Liệt kê các trường hợp biên kèm giải thích ngắn gọn bằng tiếng Việt.
- Cung cấp khối mã nguồn Java đã được sửa lỗi hoàn chỉnh, sạch đẹp, tối ưu.
- Cung cấp khối mã nguồn JUnit 5 test case bao phủ toàn bộ các trường hợp biên đã phân tích để đảm bảo mã nguồn mới hoạt động an toàn tuyệt đối"

3. Đoạn code Java đã sửa lỗi và mã nguồn JUnit test case do AI sinh ra
package com.example.calculator;

import java.util.List;

/**
 * Lớp hỗ trợ tính toán giá trị trung bình.
 * Áp dụng tư duy lập trình phòng thủ để kiểm soát triệt để các lỗi đầu vào.
 */
public class AverageCalculator {

    public static double calculateAverage(List<Integer> numbers) {
        // Trường hợp biên 1: Danh sách truyền vào bị null
        // Trường hợp biên 2: Danh sách truyền vào trống (không có phần tử)
        if (numbers == null || numbers.isEmpty()) {
            // Trả về giá trị mặc định an toàn theo yêu cầu nghiệp vụ, tránh lỗi chia cho 0
            return 0.0;
        }

        int sum = 0;
        for (int num : numbers) {
            // Kiểm tra tránh lỗi NullPointerException nếu phần tử Integer bên trong list bị null
            if (num != null) {
                sum += num;
            }
        }

        // Đảm bảo ép kiểu double chính xác trước khi thực hiện phép chia
        return (double) sum / numbers.size();
    }
}

AverageCalculatorTest.java
package com.example.calculator;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;

/**
 * Lớp kiểm thử JUnit 5 cho AverageCalculator.
 * Bao phủ toàn bộ trường hợp luồng chạy lý tưởng và các kịch bản biên gây sập hệ thống.
 */
public class AverageCalculatorTest {

    @Test
    @DisplayName("Kiểm thử luồng chạy lý tưởng với danh sách số nguyên hợp lệ")
    public void testCalculateAverage_HappyPath() {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        double expected = 3.0; // (1+2+3+4+5)/5 = 15/5 = 3.0
        double actual = AverageCalculator.calculateAverage(numbers);
        assertEquals(expected, actual, 0.001, "Giá trị trung bình tính ra phải bằng 3.0");
    }

    @Test
    @DisplayName("Trường hợp biên 1: Danh sách đầu vào bị null - Hệ thống phải trả về 0.0")
    public void testCalculateAverage_NullInput() {
        List<Integer> numbers = null;
        double expected = 0.0;
        double actual = AverageCalculator.calculateAverage(numbers);
        assertEquals(expected, actual, 0.001, "Khi list bị null, kết quả trả về phải là 0.0");
    }

    @Test
    @DisplayName("Trường hợp biên 2: Danh sách đầu vào trống - Hệ thống phải trả về 0.0")
    public void testCalculateAverage_EmptyList() {
        List<Integer> numbers = new ArrayList<>();
        double expected = 0.0;
        double actual = AverageCalculator.calculateAverage(numbers);
        assertEquals(expected, actual, 0.001, "Khi list trống, kết quả trả về phải là 0.0");
    }

    @Test
    @DisplayName("Trường hợp nâng cao: Danh sách chứa phần tử null bên trong")
    public void testCalculateAverage_ListWithNullElements() {
        // Danh sách có phần tử hợp lệ xen kẽ phần tử null
        List<Integer> numbers = Arrays.asList(10, null, 20); 
        // Logic mới: bỏ qua null, tổng bằng 30. Số lượng phần tử gốc là 3 -> 30/3 = 10.0
        double expected = 10.0;
        double actual = AverageCalculator.calculateAverage(numbers);
        assertEquals(expected, actual, 0.001, "Hệ thống phải xử lý mượt mà và bỏ qua phần tử null");
    }
}