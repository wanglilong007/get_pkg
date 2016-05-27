# get_pkg
require 'rubygems'
require 'nokogiri'
require 'open-uri'
require 'openssl'
require 'fileutils'

def down_pkg(base_url, dir, start_i)
  url = "#{base_url}#{dir}"
  start = start_i.to_i
  FileUtils.mkdir_p(dir) unless File.exists?(dir)
  option = {:ssl_verify_mode=>OpenSSL::SSL::VERIFY_NONE,
            :proxy_http_basic_authentication =>
                ["http://proxy.domain.com:8080", "user", "password"]}
  page = Nokogiri::HTML(open(url,option))
  rsc_list = page.css('tr')
  len = rsc_list.length
  rsc_list[start..len-2].each { |rsc|
    rsc_name = rsc.css('td>a').text
    p rsc_name
    if rsc_name.end_with?('/')
      down_pkg(base_url, "#{dir}#{rsc_name}", start_i)
    else
      http_file = "#{url}#{rsc_name}"
      local_file = "#{dir}#{rsc_name}"
      if !File.exists?(local_file)
        open(local_file, 'wb') do |file|
          file << open(http_file,option).read
        end
      end
    end
  }
end
down_pkg(ARGV[0], ARGV[1], ARGV[2])
