import cv2
import os
import subprocess

# ==========================
# الإعدادات
# ==========================

image_folder = "images"
temp_video = "temp.avi"
final_video = "final_tv_video.mp4"

fps = 25                  # أفضل للتلفاز
seconds_per_image = 30    # مدة كل صورة
target_width = 3840       # تحويل إلى 1080p
target_height = 2160

# ==========================
# قراءة الصور
# ==========================

images = sorted([
    img for img in os.listdir(image_folder)
    if img.lower().endswith((".png", ".jpg", ".jpeg"))
])

if not images:
    print("لا توجد صور في المجلد!")
    exit()

# ==========================
# إنشاء فيديو مؤقت بجودة عالية (بدون ضغط قوي)
# ==========================

fourcc = cv2.VideoWriter_fourcc(*'MJPG')
video = cv2.VideoWriter(temp_video, fourcc, fps, (target_width, target_height))

frames_per_image = fps * seconds_per_image

for image in images:
    image_path = os.path.join(image_folder, image)
    frame = cv2.imread(image_path)

    if frame is None:
        continue

    # تغيير الحجم إلى 1080p
    frame = cv2.resize(frame, (target_width, target_height))

    for _ in range(frames_per_image):
        video.write(frame)

video.release()
cv2.destroyAllWindows()

print("تم إنشاء الفيديو المؤقت.")

# ==========================
# ضغط الفيديو بأعلى جودة (CRF 18)
# ==========================

ffmpeg_command = [
    "ffmpeg",
    "-y",
    "-i", temp_video,
    "-c:v", "libx264",
    "-preset", "slow",
    "-crf", "18",
    "-pix_fmt", "yuv420p",
    "-profile:v", "high",
    "-level", "4.2",
    "-movflags", "+faststart",
    final_video
]

subprocess.run(ffmpeg_command)

# ==========================
# حذف الملف المؤقت
# ==========================

if os.path.exists(temp_video):
    os.remove(temp_video)

print("تم إنشاء الفيديو النهائي بجودة ممتازة جدًا ومتوافق مع التلفاز!")
