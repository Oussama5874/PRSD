import requests
import subprocess
import time

# إعدادات
ACCESS_TOKEN = "EAANZA3TeAR7QBO2GP6YCGZCkUadNJMHZCUyzspUdkdW7COWNFcjd3Cmxangn1ZA4pbEhQDi0chp8ZCfXXI8xpqxie0FGLjZBZCoje46xEJGg8K1D2yQH1TYbUlHaZCEyjcC73LMtfvTZBnKQLIJoLM6bS2pqVKYiFOdYYfX3a5opGz4PAZBd2YEsM2L7pLcZABBkCcFu8iyorXO2kejcB4e4PAymY4ZD"
PAGE_ID = "154392977758476"
M3U8_URL = "https://cdnamd-hls-globecast.akamaized.net/live/ramdisk/2m_monde/hls_video_ts_tuhawxpiemz257adfc/index.m3u8"  # رابط M3U8 الذي تريد بثه

def create_rtmp_stream(access_token, page_id):
    """
    إنشاء جلسة بث مباشر والحصول على رابط RTMP.
    """
    url = f"https://graph.facebook.com/v21.0/{page_id}/live_videos"
    payload = {
        "title": "My Live Stream",  # عنوان البث
        "description": "Streaming to Facebook Live!",  # وصف البث
        "access_token": access_token
    }

    try:
        # إرسال الطلب إلى Facebook API
        response = requests.post(url, data=payload)
        response.raise_for_status()

        # معالجة الاستجابة
        data = response.json()
        if "stream_url" in data:
            print(f"Stream URL: {data['stream_url']}")
            return data['stream_url']
        else:
            print("Failed to retrieve RTMP URL. Response:", data)
            return None
    except requests.exceptions.RequestException as e:
        print(f"Error: {e}")
        if 'response' in locals():
            print("Response Status Code:", response.status_code)
            print("Response Content:", response.text)
        return None

def start_stream(m3u8_url, rtmp_url):
    """
    بدء البث المباشر من رابط M3U8 إلى Facebook باستخدام FFmpeg.
    """
    try:
        command = [
            "ffmpeg",
            "-re",
            "-i", m3u8_url,
            "-c:v", "libx264",
            "-c:a", "aac",
            "-strict", "experimental",
            "-f", "flv",
            rtmp_url
        ]
        print("Starting the stream...")

        # تنفيذ الأمر ffmpeg
        process = subprocess.Popen(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
        stdout, stderr = process.communicate()  # تم تعديل هنا

        # طباعة المخرجات
        print("FFmpeg output (stdout):")
        print(stdout)

        if stderr:
            print("FFmpeg error (stderr):")
            print(stderr)

        # إذا نجح البث، ابدأ العد
        start_counter()

    except FileNotFoundError:
        print("FFmpeg not found. Please install FFmpeg and ensure it is in the system PATH.")
    except Exception as e:
        print(f"Unexpected error while starting stream: {e}")
        return None

def start_counter():
    """
    بدء العد من 1 إلى ما لا نهاية.
    """
    print("Broadcast started successfully. Starting counter...")
    counter = 1
    while True:
        print(f"Counter: {counter}")
        counter += 1
        time.sleep(1)  # انتظر ثانية واحدة بين كل رقم

def main():
    try:
        print("Getting RTMP URL...")
        rtmp_url = create_rtmp_stream(ACCESS_TOKEN, PAGE_ID)
        if not rtmp_url:
            print("Failed to retrieve RTMP URL.")
            return

        print("Starting Stream...")
        start_stream(M3U8_URL, rtmp_url)

    except Exception as e:
        print(f"Critical error occurred: {e}")

if __name__ == "__main__":
    main()
