##Bug задлан: Dirty pipe – CVE-2022-0847
Dirty pipe нь линуксийн kernel v5.8 хувилбар дээр гарсан дурын read-only файлыг дарж бичих боломжтой эмзэг байдал /Dirty COW-тэй төстэй/ бөгөөд мэдээж дурын файлыг дарж бичиж байгаа учраас privilege escalation буюу энгийн хэрэглэгчээс root хэрэглэгч болтол эрхээ ахиулах боломжийг олгож байгаа юм. 
2021-04-29нд CM4all веб хостинг системийг хариуцдаг Max Kellermann гэх залууд харилцагчаас нь татаж авсан access log нь gzip-ээр задрахгүй буюу эвдэрсэн байна гэсэн гомдол орж иржээ. Файлыг шалгаж үзэхэд доторх контент нь эвдрээгүй байсан ба gzip задалж чадахгүй байсан шалтгаан нь файлын CRC checksum нь алдаатай байсан учраас checksum-ыг нь гараараа засаад тухайн гомдлыг хаасан байна. Гэтэл энэ гомдлоос жилийн дараа 2022 оны 2-р сард мөн л өмнөхтэй адил шалтгаантай эвдэрсэн хэд хэдэн файлын талаарх гомдол орж ирснээр мань хүн шалтгааныг нь олохоор шинжилгээ хийх явцдаа линуксийн кернелийн түвшний эмзэг байдал болохыг олж илрүүлжээ. Dirty pipe нь зөвхөн линукс гэлтгүй андройд үйлдлийн систем дээр ч боломжтой байсан учраас маш өргөн хүрээг хамарсан эмзэг байдал юм.
За тэгэхээр яагаад ийм зүйл болсон тухай задлах юм бол линукс үйлдлийн системийн зарим concept-ыг ойлгох шаардлагатай болох юм. Линукс, windows зэрэг бүх үйлдлийн систем user mode, kernel mode гэсэн 2 хэсэгт хуваагдах бөгөөд user mode-ын процессууд нь тусгаарлалт, хамгаалалт өндөртэй мөн бага privilege-тэй байдаг бол kernel mode-д зөвхөн итгэлтэй буюу үйлдлийн системийн өөрийнх native процессууд ажиллах боловч хоорондын тусгаарлалт байхгүй учраас кернелийн түвшний эмзэг байдал хамгийн аюултай байдаг. Өөрөөр хэлбэл кернелийн түвшний эмзэг байдлыг ашиглан тухайн үйлдлийн системийг эвдэх, бүх удирдлагыг гартаа авч болно гэсэн үг. Хэрвээ user mode-оос kernel mode-руу хандах бол тусгай хамгаалалт өндөртэй интерфэйсүүдээр дамждаг.
 
Мөн үйлдлийн системд физик санах ой /RAM/ нь ашиглах хязгаартай учраас виртуал санах ойг ашигладаг ба   линуксийн виртуал санах ойн хамгийн бага нэгж нь page  бөгөөд  ихэвчлэн 4096 байт хэмжээтэй байдаг ч процессорын архитектураас хамаараад хэмжээ нь өөрчлөгдөж болно. Физик санах ойд харгалзах виртуал санах ойн мэдээллийг хадгалдаг хэсгийг page table гэнэ. Үйлдлийн систем дискнээс файл унших юм бол кернелийн түвшинд санах ойруу 4096 байтууд бүхий page-үүдийн цуваа болон ачаалагдана гэсэн үг. 
 
Кернел 4.5-аас өмнө систем дээр хэрэглэгч “File 1” нэртэй файлыг хуулан хаа нэгтээ paste хийхэд /File 2/ тухайн файлд харгалзах page-үүдийг kernel space-ээс уншаад user space дэх буферт түр хадгалаад дахин kernel space-рүү бичилт хийдэг байжээ.
 
Гэтэл кернел хөгжүүлэгчдийн дунд kernel space дээр байгаа файлыг заавал user space-ээр дамжуулаад хуулбарлаад байх шаардлага байна уу? гэсэн асуулт гарч ирсэн бөгөөд kernel v2.2-д sendfile() нэртэй syscall-ыг /процессоос кернелрүү ханддаг интерфэйс/ гаргаж ирсэн ба энэ нь файл хуулахад шууд кернелийн түвшинд хуулдаг болж нэлээд resource-г хэмнэдэг болсон байна. Үүний дараа kernel v4.5-д splice syscall-ыг гаргаж ирсэн ба энэ нь хэрвээ хэрэглэгч файлыг хуулах юм бол шинээр үүсэх файл нь original файлын санах ойн заагч байхаар шийджээ. Өөрөөр хэлбэл хэрэглэгчид файл хуулагдсан юм шиг харагдах боловч ард нь зөвхөн original файлын байрлах page-ын ойн хаяг байгаа гэж ойлгох ба үүнийг zero-copy арга гэж нэрлэнэ. 
 
Splice-ын хийж байгаа арын процесс нь File 1-ын хамгийн эхний page-ын эхлэх санах ойн хаягийг pipe—руу биччихэж байгаа юм.  Pipe-ыг энгийнээр холбогч гуурсан хоолой гээд төсөөлж болох ба гуурсны нэг үзүүрт page-ын санах ойн хаяг нөгөө талд нь File 1 байгаа бөгөөд File 2-ыг нээж байгаа юм шиг боловч үнэндээ гуурсаар дамжуулаад File 1-ыг нээж байгаа .
Харин хэрвээ хэрэглэгч File 2-д өөрчлөлт хийх юм бол кернел өмнөх шигээ файлыг хуулбарлаад өөрчлөлттэй нь хамт хадгалаад явчихдаг ба Copy on Write (COW) арга гэж нэрлэнэ. Жич: Dirty COW нь сopy on write үйлдэл дээр race condition ашиглан файлруу бичдэг эмзэг байдал юм.

 
Асуудал нь юундаа байна вэ?
 Линукс үйлдлийн системийн нь “бүх зүйл файл” гэсэн санаан дээр тулгуурладаг бөгөөд өмнө яригдсан page, pipe зэрэг ч мөн файл юм. Тэгээд файлд унших, бичих, өөрчлөх үйлдлүүд хийгдэхэд First Input First Out /FIFO/ зарчмыг баримтална. Өөрөөр хэлбэл хамгийн түрүүнд орж ирсэн файлыг боловсруулж дууссаны дараа түүний ардах файлыг боловсруулна гэх мэтээр.  
Pipe нь мөн л файл нь бол түүн дээр унших, бичих үйлдлүүд хийгдэх нь тодорхой бөгөөд үйлдлийн систем дээр pipe object-ыг pipe_inode_info бүтэц /struct/ төрөлтэйгөөр дараах бүтэцтэйгээр хэрэглэх ба дотроо pipe_buf нэртэй буферыг ашигладаг.
 
Хэрвээ тэр pipe_buf-ын бүтцийг задлаад харах юм бол нэг page-ын хэмжээтэй буюу 4096 байтын хэмжээтэй дотроо  өгөгдлийн бас pipe-ын metadata мэдээллийг агуулсан байна.
 
Хамгийн сонирхолтой нь хэрвээ pipe-руу бичилт хийх юм бол үйлдлийн систем маань бичилт хийх боломжтой pipe_buf-руу өгөгдлөө бичнэ. Гэтэл pipe_buf маань 4096 байтын багтаамжтай учраас үргүй зардал бага гаргах гээд өөр бичих зүйл байвал нэмээд дүүртэл нь хүлээх тохиолдол гардаг байна.
 
2016 онд кернелд зөвхөн anonymous pipe_buf-рүү дараалан бичсэн өгөгдлийг merge хийх боломжтой CAN_MERGE flag-ыг гаргаж ирсэн ба 2020 оны өөрчлөлтөөр PIPE_BUF_FLAG_CAN_MERGE flag-ыг бүх pipe_buf дээр тохируулах боломжтой болгосон ч default-аар огт утга оноолгүйгээр орхисон байна. 
