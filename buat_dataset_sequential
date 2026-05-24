import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'viewmodels/create_dataset_viewmodel.dart';
import 'viewmodels/auth_viewmodel.dart';
import 'models/dataset_model.dart';

class CreateDatasetPage extends StatefulWidget {
  const CreateDatasetPage({super.key});

  @override
  State<CreateDatasetPage> createState() => _CreateDatasetPageState();
}

class _CreateDatasetPageState extends State<CreateDatasetPage> {
  final _formKey = GlobalKey<FormState>();
  
  final TextEditingController _namaDatasetController = TextEditingController();
  final TextEditingController _deskripsiController = TextEditingController();
  final TextEditingController _kategoriLainnyaController = TextEditingController();
  String? _kategori;

  int currentStep = 0;

  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addPostFrameCallback((_) {
      Provider.of<CreateDatasetViewModel>(context, listen: false).reset();
    });
  }

  @override
  void dispose() {
    _namaDatasetController.dispose();
    _deskripsiController.dispose();
    _kategoriLainnyaController.dispose();
    super.dispose();
  }

  bool validateStep() {
    if (currentStep == 0) {
      if (_namaDatasetController.text.trim().isEmpty) return false;
      if (_kategori == 'Lainnya' && _kategoriLainnyaController.text.trim().isEmpty) return false;
      return true;
    }
    if (currentStep == 1) {
      // Deskripsi bersifat opsional
      return true; 
    }
    if (currentStep == 2) {
      final viewModel = Provider.of<CreateDatasetViewModel>(context, listen: false);
      if (!viewModel.hasFiles) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Tambahkan setidaknya 1 file dataset!'), backgroundColor: Colors.red),
        );
        return false;
      }
      if (!viewModel.areAllFilesUploaded) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Terdapat baris file yang belum diupload/dipilih!'), backgroundColor: Colors.red),
        );
        return false;
      }
      return true;
    }
    return true; // Step 3 (Summary) selalu valid
  }

  void nextStep() {
    if (!validateStep()) {
      if (currentStep == 0) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Lengkapi form yang wajib diisi pada tahap ini.'), backgroundColor: Colors.red),
        );
      }
      return;
    }

    setState(() {
      if (currentStep < 3) {
        currentStep++;
      }
    });
  }

  void previousStep() {
    setState(() {
      if (currentStep > 0) {
        currentStep--;
      }
    });
  }

  void _simpanData(CreateDatasetViewModel viewModel) {
    if (!_formKey.currentState!.validate()) return;
    
    final String namaDataset = _namaDatasetController.text;
    
    final authVM = Provider.of<AuthViewModel>(context, listen: false);
    final ownerName = authVM.currentUser?.name ?? 'General';
    final DatasetModel newDataset = viewModel.generateDataset(namaDataset, ownerName);

    Navigator.pop(context, newDataset);
    
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Dataset "$namaDataset" beserta ${viewModel.fileCount} file berhasil disimpan!')),
    );
  }

  void _onCategoryChanged(String? val) {
    setState(() {
      _kategori = val;
    });
  }

  Widget get activeFragment {
    final viewModel = Provider.of<CreateDatasetViewModel>(context);
    const Color primaryRed = Color.fromARGB(247, 170, 28, 28);

    if (currentStep == 0) {
      return BasicInfoFragment(
        namaDatasetController: _namaDatasetController,
        kategoriLainnyaController: _kategoriLainnyaController,
        kategori: _kategori,
        onCategoryChanged: _onCategoryChanged,
        primaryColor: primaryRed,
      );
    }
    if (currentStep == 1) {
      return DescriptionFragment(
        deskripsiController: _deskripsiController,
        primaryColor: primaryRed,
      );
    }
    if (currentStep == 2) {
      return FileDataFragment(
        viewModel: viewModel,
        primaryColor: primaryRed,
      );
    }
    return SummaryFragment(
      namaDatasetController: _namaDatasetController,
      deskripsiController: _deskripsiController,
      kategoriLainnyaController: _kategoriLainnyaController,
      kategori: _kategori,
      viewModel: viewModel,
    );
  }

  @override
  Widget build(BuildContext context) {
    const Color primaryRed = Color.fromARGB(247, 170, 28, 28);

    return Scaffold(
      appBar: AppBar(
        title: const Text(
          'Buat Dataset Baru', 
          style: TextStyle(color: primaryRed, fontWeight: FontWeight.bold)
        ),
        backgroundColor: Colors.white,
        iconTheme: const IconThemeData(color: primaryRed),
        elevation: 0,
        centerTitle: true,
      ),
      backgroundColor: Colors.grey[100],
      body: Form(
        key: _formKey,
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 24.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              // 1. Progress Indicator
              LinearProgressIndicator(
                value: (currentStep + 1) / 4,
                backgroundColor: Colors.grey[300],
                valueColor: const AlwaysStoppedAnimation<Color>(primaryRed),
                minHeight: 8,
                borderRadius: BorderRadius.circular(4),
              ),
              const SizedBox(height: 8),
              Text(
                'Langkah ${currentStep + 1} dari 4',
                textAlign: TextAlign.end,
                style: const TextStyle(fontWeight: FontWeight.w500),
              ),
              const SizedBox(height: 16),
              
              // 2. Fragment Aktif
              Expanded(
                child: SingleChildScrollView(
                  child: activeFragment,
                ),
              ),
              const SizedBox(height: 16),
              
              // 3. Tombol Navigasi Bawah
              Row(
                children: [
                  Expanded(
                    child: OutlinedButton(
                      onPressed: currentStep == 0 ? () => Navigator.pop(context) : previousStep,
                      style: OutlinedButton.styleFrom(
                        foregroundColor: primaryRed,
                        side: const BorderSide(color: primaryRed, width: 1.5),
                        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
                        padding: const EdgeInsets.symmetric(vertical: 16),
                      ),
                      child: Text(currentStep == 0 ? 'Batal' : 'Kembali', style: const TextStyle(fontWeight: FontWeight.bold)),
                    ),
                  ),
                  const SizedBox(width: 12),
                  Expanded(
                    child: ElevatedButton(
                      onPressed: currentStep == 3 ? () {
                        final vm = Provider.of<CreateDatasetViewModel>(context, listen: false);
                        _simpanData(vm);
                      } : nextStep,
                      style: ElevatedButton.styleFrom(
                        backgroundColor: primaryRed,
                        foregroundColor: Colors.white,
                        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
                        elevation: 0,
                        padding: const EdgeInsets.symmetric(vertical: 16),
                      ),
                      child: Text(currentStep == 3 ? 'Simpan Dataset' : 'Lanjut', style: const TextStyle(fontWeight: FontWeight.bold)),
                    ),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}


class BasicInfoFragment extends StatelessWidget {
  final TextEditingController namaDatasetController;
  final TextEditingController kategoriLainnyaController;
  final String? kategori;
  final ValueChanged<String?> onCategoryChanged;
  final Color primaryColor;

  const BasicInfoFragment({
    super.key,
    required this.namaDatasetController,
    required this.kategoriLainnyaController,
    required this.kategori,
    required this.onCategoryChanged,
    required this.primaryColor,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 0,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12), side: BorderSide(color: Colors.grey[300]!)),
      color: Colors.white,
      child: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Center(
              child: Text('Informasi Dasar', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            ),
            const SizedBox(height: 24),
            const Text('Nama Dataset', style: TextStyle(fontWeight: FontWeight.w600, fontSize: 13)),
            const SizedBox(height: 8),
            TextFormField(
              controller: namaDatasetController,
              decoration: _inputDeco('Masukkan nama dataset', primaryColor),
              validator: (val) => val == null || val.isEmpty ? 'Nama dataset wajib diisi' : null,
            ),
            const SizedBox(height: 20),
            const Text('Kategori', style: TextStyle(fontWeight: FontWeight.w600, fontSize: 13)),
            const SizedBox(height: 8),
            DropdownButtonFormField<String>(
              value: kategori,
              hint: Text('Pilih Kategori', style: TextStyle(color: Colors.grey[400], fontSize: 14)),
              icon: const Icon(Icons.keyboard_arrow_down),
              decoration: _inputDeco('', primaryColor),
              items: ['Kesehatan', 'Pendidikan', 'Ekonomi', 'Teknologi', 'Lainnya']
                  .map((k) => DropdownMenuItem(value: k, child: Text(k, style: const TextStyle(fontSize: 14))))
                  .toList(),
              onChanged: onCategoryChanged,
            ),
            if (kategori == 'Lainnya') ...[
              const SizedBox(height: 12),
              const Text('Nama Kategori Baru', style: TextStyle(fontWeight: FontWeight.w600, fontSize: 13)),
              const SizedBox(height: 8),
              TextFormField(
                controller: kategoriLainnyaController,
                decoration: _inputDeco('Ketik kategori di sini...', primaryColor),
                validator: (val) => val == null || val.isEmpty ? 'Kategori baru wajib diisi' : null,
              ),
            ],
          ],
        ),
      ),
    );
  }

  InputDecoration _inputDeco(String hint, Color focusColor) {
    return InputDecoration(
      hintText: hint,
      hintStyle: TextStyle(color: Colors.grey[400], fontSize: 14),
      border: OutlineInputBorder(borderRadius: BorderRadius.circular(8), borderSide: BorderSide(color: Colors.grey[300]!)),
      enabledBorder: OutlineInputBorder(borderRadius: BorderRadius.circular(8), borderSide: BorderSide(color: Colors.grey[300]!)),
      focusedBorder: OutlineInputBorder(borderRadius: BorderRadius.circular(8), borderSide: BorderSide(color: focusColor)),
      contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
    );
  }
}

class DescriptionFragment extends StatelessWidget {
  final TextEditingController deskripsiController;
  final Color primaryColor;

  const DescriptionFragment({
    super.key,
    required this.deskripsiController,
    required this.primaryColor,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 0,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12), side: BorderSide(color: Colors.grey[300]!)),
      color: Colors.white,
      child: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Center(
              child: Text('Detail Deskripsi', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            ),
            const SizedBox(height: 24),
            const Text('Deskripsi (Opsional)', style: TextStyle(fontWeight: FontWeight.w600, fontSize: 13)),
            const SizedBox(height: 8),
            TextFormField(
              controller: deskripsiController,
              maxLines: 6,
              decoration: InputDecoration(
                hintText: 'Tambahkan deskripsi detail tentang dataset ini...',
                hintStyle: TextStyle(color: Colors.grey[400], fontSize: 14),
                border: OutlineInputBorder(borderRadius: BorderRadius.circular(8), borderSide: BorderSide(color: Colors.grey[300]!)),
                enabledBorder: OutlineInputBorder(borderRadius: BorderRadius.circular(8), borderSide: BorderSide(color: Colors.grey[300]!)),
                focusedBorder: OutlineInputBorder(borderRadius: BorderRadius.circular(8), borderSide: BorderSide(color: primaryColor)),
                contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class FileDataFragment extends StatelessWidget {
  final CreateDatasetViewModel viewModel;
  final Color primaryColor;

  const FileDataFragment({
    super.key,
    required this.viewModel,
    required this.primaryColor,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 0,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12), side: BorderSide(color: Colors.grey[300]!)),
      color: Colors.white,
      child: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                const Text('File Dataset', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                ElevatedButton.icon(
                  onPressed: () => viewModel.tambahFile(),
                  icon: const Icon(Icons.add, size: 16),
                  label: const Text('Tambah Baris', style: TextStyle(fontSize: 12, fontWeight: FontWeight.bold)),
                  style: ElevatedButton.styleFrom(
                    backgroundColor: primaryColor,
                    foregroundColor: Colors.white,
                    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
                    padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 10),
                    elevation: 0,
                  ),
                ),
              ],
            ),
            const SizedBox(height: 20),
            Container(height: 2, color: primaryColor),
            
            if (!viewModel.hasFiles)
               const Padding(
                 padding: EdgeInsets.symmetric(vertical: 32.0),
                 child: Center(
                   child: Text(
                     'Belum ada file.\nKlik "Tambah Baris" untuk upload data.', 
                     textAlign: TextAlign.center,
                     style: TextStyle(color: Colors.grey)
                   )
                 ),
               )
            else
              ListView.builder(
                shrinkWrap: true,
                physics: const NeverScrollableScrollPhysics(),
                itemCount: viewModel.fileCount,
                itemBuilder: (context, index) {
                  final file = viewModel.fileList[index];
                  return Container(
                    decoration: const BoxDecoration(
                      border: Border(bottom: BorderSide(color: Colors.black12))
                    ),
                    padding: const EdgeInsets.symmetric(vertical: 12.0),
                    child: Row(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Expanded(
                          flex: 4, 
                          child: Padding(
                            padding: const EdgeInsets.only(right: 8.0),
                            child: InkWell(
                              onTap: () => viewModel.pilihFile(index),
                              child: Container(
                                padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 14),
                                decoration: BoxDecoration(
                                  border: Border.all(color: Colors.grey[400]!),
                                  borderRadius: BorderRadius.circular(6),
                                  color: file.namaFile.isEmpty ? const Color.fromARGB(255, 250, 250, 250) : const Color.fromARGB(255, 230, 245, 230)
                                ),
                                child: Row(
                                  children: [
                                    Icon(file.namaFile.isEmpty ? Icons.upload_file : Icons.check_circle, size: 16, color: file.namaFile.isEmpty ? Colors.grey : Colors.green),
                                    const SizedBox(width: 8),
                                    Expanded(
                                      child: Text(
                                        file.namaFile.isEmpty ? 'Pilih File (CSV/Excel)' : file.namaFile,
                                        style: file.namaFile.isEmpty ? const TextStyle(color: Colors.grey, fontSize: 13) : const TextStyle(color: Colors.black, fontSize: 13, fontWeight: FontWeight.w500),
                                        maxLines: 1,
                                        overflow: TextOverflow.ellipsis,
                                      ),
                                    ),
                                  ],
                                ),
                              ),
                            ),
                          ),
                        ),
                        Expanded(
                          flex: 4, 
                          child: Padding(
                            padding: const EdgeInsets.only(right: 8.0),
                            child: TextFormField(
                              initialValue: file.description,
                              style: const TextStyle(fontSize: 13),
                              decoration: InputDecoration(
                                hintText: 'Tulis Deskripsi',
                                isDense: true,
                                contentPadding: const EdgeInsets.symmetric(horizontal: 12, vertical: 14),
                                border: OutlineInputBorder(borderRadius: BorderRadius.circular(6)),
                              ),
                              onChanged: (val) => viewModel.ubahDeskripsi(index, val),
                            ),
                          ),
                        ),
                        SizedBox(
                          width: 40,
                          child: IconButton(
                            onPressed: () => viewModel.hapusFile(index),
                            icon: const Icon(Icons.delete_outline, color: Colors.red),
                            tooltip: 'Hapus Baris',
                            padding: EdgeInsets.zero,
                            constraints: const BoxConstraints(),
                          ),
                        ),
                      ],
                    ),
                  );
                },
              ),
          ],
        ),
      ),
    );
  }
}

class SummaryFragment extends StatelessWidget {
  final TextEditingController namaDatasetController;
  final TextEditingController deskripsiController;
  final TextEditingController kategoriLainnyaController;
  final String? kategori;
  final CreateDatasetViewModel viewModel;

  const SummaryFragment({
    super.key,
    required this.namaDatasetController,
    required this.deskripsiController,
    required this.kategoriLainnyaController,
    required this.kategori,
    required this.viewModel,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 0,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12), side: BorderSide(color: Colors.grey[300]!)),
      color: Colors.white,
      child: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Center(
              child: Text('Ringkasan Data', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            ),
            const SizedBox(height: 24),
            _buildSummaryRow('Nama Dataset', namaDatasetController.text),
            _buildSummaryRow('Kategori', kategori == 'Lainnya' ? kategoriLainnyaController.text : (kategori ?? '-')),
            _buildSummaryRow('Deskripsi', deskripsiController.text.isEmpty ? '-' : deskripsiController.text),
            _buildSummaryRow('Total File', '${viewModel.fileCount} file(s)'),
            const SizedBox(height: 16),
            const Text('Daftar File:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            ...viewModel.fileList.map((f) => Padding(
              padding: const EdgeInsets.only(bottom: 8.0, left: 8.0),
              child: Row(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Icon(Icons.insert_drive_file, size: 16, color: Colors.grey),
                  const SizedBox(width: 8),
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(f.namaFile, style: const TextStyle(fontWeight: FontWeight.w600, fontSize: 13)),
                        if (f.description.isNotEmpty)
                          Text(f.description, style: const TextStyle(color: Colors.grey, fontSize: 12)),
                      ],
                    ),
                  ),
                ],
              ),
            )).toList(),
          ],
        ),
      ),
    );
  }

  Widget _buildSummaryRow(String label, String value) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 12.0),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          SizedBox(width: 120, child: Text(label, style: const TextStyle(color: Colors.grey, fontSize: 13))),
          Expanded(child: Text(value, style: const TextStyle(fontWeight: FontWeight.w600, fontSize: 13))),
        ],
      ),
    );
  }
}
