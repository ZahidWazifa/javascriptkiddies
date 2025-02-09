# PICO_CTF WRITEUP-javascriptkiddies
![image](https://github.com/user-attachments/assets/f7ecb6bc-9e7f-45f3-94b2-780b2ab68e96)

at first glance I saw information on the input form where after I saw a new file appeared, 
![image](https://github.com/user-attachments/assets/3b05ccec-ad93-4e04-9674-a86dc6426ee7)

however, after inspecting the webpage I found a file that handles files that are thrown to users.
```js
			var bytes = [];
			$.get("bytes", function(resp) {
				bytes = Array.from(resp.split(" "), x => Number(x));
			});

			function assemble_png(u_in){
				var LEN = 16;
				var key = "0000000000000000";
				var shifter;
				if(u_in.length == LEN){
					key = u_in;
				}
				var result = [];
				for(var i = 0; i < LEN; i++){
					shifter = key.charCodeAt(i) - 48;
					for(var j = 0; j < (bytes.length / LEN); j ++){
						result[(j * LEN) + i] = bytes[(((j + shifter) * LEN) % bytes.length) + i]
					}
				}
				while(result[result.length-1] == 0){
					result = result.slice(0,result.length-1);
				}
				document.getElementById("Area").src = "data:image/png;base64," + btoa(String.fromCharCode.apply(null, new Uint8Array(result)));
				return false;
			}
		
```
if you look carefully at the code, you can see that the code has keys that will be accepted on the server if it is correct, with the code `0000000000000000` based on this I tried to enter it into the form and when using burp suite i get the response form the server that give this resposense:
![image](https://github.com/user-attachments/assets/b980df7c-e84c-4675-9829-3906aed68558)
if you look closely at the file you can see that there are a lot of bytes, from the initial conclusion of how the browser is trying to send the image file, this narrows down the possibility that there is a possibility that the file is between png, jpg, jpeg, svg or other formats. based on the code that appears: 
```png
128 252 182 115 177 211 142 252 189 248 130 93 154 0 68 90 131 255 204 170 239 167 18 51 233 43 0 26 210 72 95 120 227 7 195 126 207 254 115 53 141 217 0 11 118 192 110 0 0 170 248 73 103 78 10 174 208 233 156 187 185 65 228 0 137 128 228 71 159 10 111 10 29 96 71 238 141 86 91 82 0 214 37 114 7 0 238 114 133 0 140 0 38 36 144 108 164 141 63 2 69 73 15 65 68 0 249 13 0 64 111 220 48 0 55 255 13 12 68 41 66 120 188 0 73 27 173 72 189 80 0 148 0 64 26 123 0 32 44 237 0 252 36 19 52 0 78 227 98 88 1 185 1 128 182 177 155 44 132 162 68 0 1 239 175 248 68 91 84 18 223 223 111 83 26 188 241 12 0 197 57 89 116 96 223 96 161 45 133 127 125 63 80 129 69 59 241 157 0 105 57 23 30 241 62 229 128 91 39 152 125 146 216 91 5 217 16 48 159 4 198 23 108 178 199 14 6 175 51 154 227 45 56 140 221 0 230 228 99 239 132 198 133 72 243 93 3 86 94 246 156 153 123 1 204 200 233 143 127 64 164 203 36 24 2 169 121 122 159 40 4 25 64 0 241 9 94 220 254 221 122 8 22 227 140 221 248 250 141 66 78 126 190 73 248 105 5 14 26 19 119 223 103 165 69 177 68 61 195 239 115 199 126 61 41 242 175 85 211 11 5 250 93 79 194 78 245 223 255 189 0 128 9 150 178 0 112 247 210 21 36 0 2 252 144 59 101 164 185 94 232 59 150 255 187 1 198 171 182 228 147 73 149 47 92 133 147 254 173 242 39 254 223 214 196 135 248 34 146 206 63 127 127 22 191 92 88 69 23 142 167 237 248 23 215 148 166 59 243 248 173 210 169 254 209 157 174 192 32 228 41 192 245 47 207 120 139 28 224 249 29 55 221 109 226 21 129 75 41 113 192 147 45 144 55 228 126 250 127 197 184 155 251 19 220 11 241 171 229 213 79 135 93 49 94 144 38 250 121 113 58 114 77 111 157 146 242 175 236 185 60 67 173 103 233 234 60 248 27 242 115 223 207 218 203 115 47 252 241 152 24 165 115 126 48 76 104 126 42 225 226 211 57 252 239 21 195 205 107 255 219 132 148 81 171 53 79 91 27 174 235 124 213 71 221 243 212 38 224 124 54 77 248 252 88 163 44 191 109 63 189 231 251 189 242 141 246 249 15 0 2 230 7 244 161 31 42 182 219 15 221 164 252 207 53 95 99 60 190 232 78 255 197 16 169 252 100 164 19 158 32 189 126 140 145 158 116 245 68 94 149 111 252 74 135 189 83 74 71 218 99 220 208 87 24 228 11 111 245 1 0 98 131 46 22 94 71 244 22 147 21 83 155 252 243 90 24 59 73 247 223 127 242 183 251 124 28 245 222 199 248 122 204 230 79 219 147 11 225 202 239 24 132 55 89 221 143 151 137 63 150 79 211 8 16 4 60 63 99 65 0 2
```
from the number of bits that appear I tried to calculate the number of initial bits that there are a total of 16 initial bits which in the first 8 bits usually indicates that the file is a png, this reference I got from [here](http://www.libpng.org/pub/png/spec/1.2/PNG-Structure.html) png itself consists of a collection of bytes starting with an initial 8 bit hexadecimal value that starts with `137 80 78 71 13 10 26 10` and then is filled in by other values. knowing this I tried to find in these bytes whether there is a png header or not.
from the webpage also shows that the file is corrupted, to solve this I tried shifting the png chunk using a python script to get the flag
the script is here 
```python
from PIL import Image
import itertools
import io

class PNGExtractor:
    def __init__(self):
        self.KEY_LEN = 16
        self.PNG_HEADER = [0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A, 
                          0x00, 0x00, 0x00, 0x0D, 0x49, 0x48, 0x44, 0x52]
        
    def decode_bytes(self, byte_string):
        """Convert space-separated byte string to integer list"""
        return [int(x) for x in byte_string.strip().split()]
        
    def find_possible_shifts(self, bytes_arr):
        """Find possible shift values for each position"""
        shifters = [[] for _ in range(self.KEY_LEN)]
        
        for pos in range(self.KEY_LEN):
            for shift in range(10):  # Testing shifts 0-9
                offset = (shift * self.KEY_LEN + pos) % len(bytes_arr)
                if bytes_arr[offset] == self.PNG_HEADER[pos]:
                    shifters[pos].append(shift)
        
        return shifters
    
    def apply_shift_pattern(self, bytes_arr, shift_pattern):
        """Apply a shift pattern to the bytes array"""
        result = [0] * len(bytes_arr)
        for i in range(self.KEY_LEN):
            shift = int(shift_pattern[i])
            for j in range(len(bytes_arr) // self.KEY_LEN):
                src_idx = (((j + shift) * self.KEY_LEN) % len(bytes_arr)) + i
                dst_idx = (j * self.KEY_LEN) + i
                if dst_idx < len(bytes_arr):
                    result[dst_idx] = bytes_arr[src_idx]
        return result
    
    def try_create_png(self, bytes_arr, shift_pattern):
        """Try to create a PNG file with given shift pattern"""
        decoded_bytes = self.apply_shift_pattern(bytes_arr, shift_pattern)
        img_bytes = io.BytesIO(bytes(decoded_bytes))
        
        try:
            img = Image.open(img_bytes)
            output_file = f"decoded_{shift_pattern}.png"
            img.save(output_file)
            print(f"✓ Berhasil membuat: {output_file}")
            return True
        except IOError:
            print(f"✗ Pattern tidak valid: {shift_pattern}")
            return False
    
    def extract_png(self, byte_string):
        """Main function to extract PNG from byte string"""
        bytes_arr = self.decode_bytes(byte_string)
        shifters = self.find_possible_shifts(bytes_arr)
        
        successful = 0
        total = 0
        
        for shift_pattern in itertools.product(*shifters):
            total += 1
            pattern_str = ''.join(str(n) for n in shift_pattern)
            if self.try_create_png(bytes_arr, pattern_str):
                successful += 1
                
        print(f"\nProses selesai:")
        print(f"- Total percobaan kombinasi: {total}")
        print(f"- Berhasil mengekstrak: {successful} file PNG")

# Byte data yang diberikan
byte_data = """128 252 182 115 177 211 142 252 189 248 130 93 154 0 68 90 131 255 204 170 239 167 18 51 233 43 0 26 210 72 95 120 227 7 195 126 207 254 115 53 141 217 0 11 118 192 110 0 0 170 248 73 103 78 10 174 208 233 156 187 185 65 228 0 137 128 228 71 159 10 111 10 29 96 71 238 141 86 91 82 0 214 37 114 7 0 238 114 133 0 140 0 38 36 144 108 164 141 63 2 69 73 15 65 68 0 249 13 0 64 111 220 48 0 55 255 13 12 68 41 66 120 188 0 73 27 173 72 189 80 0 148 0 64 26 123 0 32 44 237 0 252 36 19 52 0 78 227 98 88 1 185 1 128 182 177 155 44 132 162 68 0 1 239 175 248 68 91 84 18 223 223 111 83 26 188 241 12 0 197 57 89 116 96 223 96 161 45 133 127 125 63 80 129 69 59 241 157 0 105 57 23 30 241 62 229 128 91 39 152 125 146 216 91 5 217 16 48 159 4 198 23 108 178 199 14 6 175 51 154 227 45 56 140 221 0 230 228 99 239 132 198 133 72 243 93 3 86 94 246 156 153 123 1 204 200 233 143 127 64 164 203 36 24 2 169 121 122 159 40 4 25 64 0 241 9 94 220 254 221 122 8 22 227 140 221 248 250 141 66 78 126 190 73 248 105 5 14 26 19 119 223 103 165 69 177 68 61 195 239 115 199 126 61 41 242 175 85 211 11 5 250 93 79 194 78 245 223 255 189 0 128 9 150 178 0 112 247 210 21 36 0 2 252 144 59 101 164 185 94 232 59 150 255 187 1 198 171 182 228 147 73 149 47 92 133 147 254 173 242 39 254 223 214 196 135 248 34 146 206 63 127 127 22 191 92 88 69 23 142 167 237 248 23 215 148 166 59 243 248 173 210 169 254 209 157 174 192 32 228 41 192 245 47 207 120 139 28 224 249 29 55 221 109 226 21 129 75 41 113 192 147 45 144 55 228 126 250 127 197 184 155 251 19 220 11 241 171 229 213 79 135 93 49 94 144 38 250 121 113 58 114 77 111 157 146 242 175 236 185 60 67 173 103 233 234 60 248 27 242 115 223 207 218 203 115 47 252 241 152 24 165 115 126 48 76 104 126 42 225 226 211 57 252 239 21 195 205 107 255 219 132 148 81 171 53 79 91 27 174 235 124 213 71 221 243 212 38 224 124 54 77 248 252 88 163 44 191 109 63 189 231 251 189 242 141 246 249 15 0 2 230 7 244 161 31 42 182 219 15 221 164 252 207 53 95 99 60 190 232 78 255 197 16 169 252 100 164 19 158 32 189 126 140 145 158 116 245 68 94 149 111 252 74 135 189 83 74 71 218 99 220 208 87 24 228 11 111 245 1 0 98 131 46 22 94 71 244 22 147 21 83 155 252 243 90 24 59 73 247 223 127 242 183 251 124 28 245 222 199 248 122 204 230 79 219 147 11 225 202 239 24 132 55 89 221 143 151 137 63 150 79 211 8 16 4 60 63 99 65 0 2"""

# Jalankan ekstraksi
extractor = PNGExtractor()
extractor.extract_png(byte_data)
```
the next step I used the zbarimg tool for scanning barcodes with the following flag results:
`picoCTF{7b15cfb95f05286e37a22dda25935e1e}`

