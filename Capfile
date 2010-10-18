###
# Capfile to pull down data from 
# to an (eu-west-1) EBS volume and snapshot it.

require 'catpaws'

set :aws_access_key,  ENV['AMAZON_ACCESS_KEY']
set :aws_secret_access_key , ENV['AMAZON_SECRET_ACCESS_KEY']
set :ec2_url, ENV['EC2_URL']
set :ebs_zone, ENV['EBS_AVAILABILITY_ZONE']
set :ssh_options, { :user => "ubuntu", :keys=>[ENV['EC2_KEYFILE']]}
set :nhosts, 1
set :key, ENV['EC2_KEY']
set :key_file, ENV['EC2_KEYFILE']
set :ami, 'ami-cf4d67bb'  #EC2 eu-west-1 
set :instance_type, 'm1.small'
set :group_name, 'mikkelsen07data'
set :dev, '/dev/sdf'
set :mount_point, '/data'

set :ebs_size, 10
set :ebs_zone, 'eu-west-1a' 

vol_id = `cat VOLUMEID`.chomp
set :vol_id, vol_id 

set :ftp_dir,  "ftp://ftp.ncbi.nlm.nih.gov/sra/static"

# Raw data from the paper:
set :geo, 'GSE12241'


  # Compressed, these files are around 1.5GB per experiment
  # getting these onto the EBS may take a while...
  # assuming de-compressed they're around 4GB per experiment, then a total of 21*4 =  84GB 
  # Say we make a 100GB EBS volume, then it's around $10 a month to store it. 

  # No time to get everything yet - will update later. SNAPID will always be uptodate.
  # for not, just get the relevant mef data so prob 10GB is ok.
  #MEF_H3K4me3_ChIPSeq = 0 
  #MEF_H3K27me3_ChIPSeq = 11

  set :datasets,  [
                   { :geo => 'GSM307608', :name => 'MEF_H3K4me3_ChIPSeq' ,   :sra => 'SRX001/SRX001933' },
                   { :geo => 'GSM307611', :name => 'MEF_H3K9me3_ChIPSeq',    :sra => 'SRX001/SRX001934' },
                   { :geo => 'GSM307614', :name => 'NP_H3K27me3_ChIPSeq',    :sra => 'SRX001/SRX001936' },
                   { :geo => 'GSM307617', :name => 'NP_WCE_ChIPSeq',         :sra => 'SRX001/SRX001940' },
                   { :geo => 'GSM307624', :name => 'ES_H3_ChIPSeq',          :sra => 'SRX001/SRX001920' },
                   { :geo => 'GSM307607', :name => 'ESHyb_H3K9me3_ChIPSeq',  :sra => 'SRX001/SRX001930' },
                   { :geo => 'GSM307610', :name => 'MEF_H3K36me3_ChIPSeq',   :sra => 'SRX001/SRX001932' },
                   { :geo => 'GSM307613', :name => 'NP_H3K4me3_ChIPSeq',     :sra => 'SRX001/SRX001938' },
                   { :geo => 'GSM307620', :name => 'ES_H3K36me3_ChIPSeq',    :sra => 'SRX001/SRX001922' },
                   { :geo => 'GSM307623', :name => 'ES_RPol2_ChIPSeq',       :sra => 'SRX001/SRX001926' },
                   { :geo => 'GSM307606', :name => 'ESHyb_H3K36me3_ChIPSeq', :sra => 'SRX001/SRX001928' },
                   { :geo => 'GSM307609', :name => 'MEF_H3K27me3_ChIPSeq',   :sra => 'SRX001/SRX001931' }
                   { :geo => 'GSM307616', :name => 'NP_H3K9me3_ChIPSeq',     :sra => 'SRX001/SRX001939' },
                   { :geo => 'GSM307619', :name => 'ES_H3K27me3_ChIPSeq',    :sra => 'SRX001/SRX001921' },
                   { :geo => 'GSM307622', :name => 'ES_H4K20me3_ChIPSeq',    :sra => 'SRX001/SRX001925' },
                   { :geo => 'GSM307625', :name => 'ES_WCE_ChIPSeq',         :sra => 'SRX001/SRX001927' },
                   { :geo => 'GSM307605', :name => 'ESHyb_H3K4me3_ChIPSeq',  :sra => 'SRX001/SRX001929' },
                   { :geo => 'GSM307612', :name => 'MEF_WCE_ChIPSeq',        :sra => 'SRX001/SRX001935' },
                   { :geo => 'GSM307615', :name => 'NP_H3K36me3_ChIPSeq',    :sra => 'SRX001/SRX001937' },
                   { :geo => 'GSM307618', :name => 'ES_H3K4me3_ChIPSeq',     :sra => 'SRX001/SRX001923' },
                   { :geo => 'GSM307621', :name => 'ES_H3K9me3_ChIPSeq',     :sra => 'SRX001/SRX001924' }
                  ]


  #before getting to here, you should run
  #cap EC2:start
  #cap EBS:create
  #cap EBS:attach
  #cap EBS:format_xfs
  #cap EBS:mount_xfs


desc "Download each dataset one to the new EBS volume"
task :fetch_data, :roles => proc{fetch :group_name} do
  datasets.each do |d|
    ftp_path =  "#{ftp_dir}/#{d[:sra]}"
    run "cd #{mount_point} && wget -r #{ftp_path}/"
  end
  run "cd #{mount_point} && mv ftp.ncbi.nlm.nih.gov/sra/static/* ."
  run "cd #{mount_point} && rm -rf ftp.ncbi.nlm.nih.gov"

end
before 'fetch_data', 'EC2:start'

desc "unpack data"
task :unpack_data, :roles => proc{fetch :group_name} do
  datasets.each do |d|
    run "cd #{mount_point}/#{d[:sra]} && bunzip2 *.bz2"
    run "cd #{mount_point} && mv #{d[:sra]} #{d[:name]}"
  end
  run "cd #{mount_point} && rmdir SRX001"
end
before 'unpack_data', 'EC2:start'

#and once that's done

# cap EBS:snapshot
# cap EBS:unmount
# cap EBS:detach
# cap EBS:delete
# cap EC2:stop
