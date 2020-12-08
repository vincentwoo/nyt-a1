require 'dotenv/load'
require 'time'
require 'http'
require 'rmagick'
require 'google/cloud/storage'
require 'tempfile'

task :download_nyt_a1 do
  date = (Time.now.utc + Time.zone_offset('EST')).to_date
  response = HTTP.get "https://static01.nyt.com/images/#{date.strftime('%Y/%m/%d')}/nytfrontpage/scan.pdf"
  if response.code == 404
    date = date.prev_day
    response = HTTP.get "https://static01.nyt.com/images/#{date.strftime('%Y/%m/%d')}/nytfrontpage/scan.pdf"
  end

  image = Magick::Image.from_blob(response.body.to_s) { self.density= '150' }[0]
  image.trim! true
  image.resize_to_fit! 1440, 2560
  image = image.extent 1440, 2560, -[(1440 - image.columns) / 2, 0].max, 0
  image = image.quantize 256, Magick::GRAYColorspace

  file = Tempfile.new([ 'nyta1', '.png' ])
  image.write file.path

  storage = Google::Cloud::Storage.new
  bucket = storage.bucket 'nyt-a1'
  bucket.create_file file, 'nyta1.png'
end
