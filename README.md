# DoAn-J2EE
1. Chuẩn bị Extension (Để dùng UUID)
Chạy lệnh này đầu tiên để PostgreSQL hiểu hàm sinh mã uuid_generate_v4():
SQL
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
2. Tạo hệ thống bảng (Schema)
SQL
-- 1. Bảng Người dùng
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    student_code VARCHAR(20) UNIQUE,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(20) CHECK (role IN ('STUDENT','ADMIN')),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Bảng Tầng (Quy hoạch vùng)
CREATE TABLE floors (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(50) NOT NULL,
    description TEXT,
    is_silent_zone BOOLEAN DEFAULT false
);

-- 3. Bảng Bàn
CREATE TABLE library_tables (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    floor_id UUID NOT NULL REFERENCES floors(id) ON DELETE CASCADE,
    table_code VARCHAR(20) NOT NULL,
    capacity INT DEFAULT 4,
    position_x INT, 
    position_y INT
);

-- 4. Bảng Ghế
CREATE TABLE seats (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    table_id UUID NOT NULL REFERENCES library_tables(id) ON DELETE CASCADE,
    seat_code VARCHAR(20) NOT NULL,
    has_power_outlet BOOLEAN DEFAULT false,
    is_damaged BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 5. Bảng Đặt chỗ (Logic chính)
CREATE TABLE bookings (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users(id),
    seat_id UUID NOT NULL REFERENCES seats(id),
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    status VARCHAR(20) CHECK (status IN ('BOOKED', 'CHECKED_IN', 'COMPLETED', 'CANCELLED', 'EXPIRED')) DEFAULT 'BOOKED',
    checkin_time TIMESTAMP,
    checkout_time TIMESTAMP,
    qr_token VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
-- 6. Bảng Cấu hình hệ thống (Để quản lý Quota 4h/ngày)
CREATE TABLE system_configs (
    config_key VARCHAR(50) PRIMARY KEY,
    config_value VARCHAR(255) NOT NULL,
    description TEXT
);
3. Tạo Index (Để truy vấn nhanh hơn - Ghi điểm với thầy)
Chạy các lệnh này để tối ưu tốc độ tìm kiếm ghế trống:
SQL
CREATE INDEX idx_booking_time_range ON bookings(start_time, end_time);
CREATE INDEX idx_booking_status_filter ON bookings(status);
CREATE INDEX idx_seat_table_map ON seats(table_id);
	Thêm ràng buộc để tránh "Ghế trùng mã"
Trong cùng một bàn, không nên có hai ghế trùng tên (ví dụ: bàn 1 không thể có hai ghế cùng tên 'A').
•	Sửa đổi: Bạn nên thêm một ràng buộc UNIQUE cho cặp table_id và seat_code.
•	Lệnh SQL bổ sung:
SQL
ALTER TABLE seats ADD CONSTRAINT unique_seat_per_table UNIQUE (table_id, seat_code);

