// context/AppContext.js
import { createContext } from 'react';

export const AppContext = createContext();

// screens/EntryScreen.js
import React, { useState, useContext } from 'react';
import { 
  View, 
  Text, 
  StyleSheet, 
  ScrollView, 
  TextInput, 
  TouchableOpacity, 
  Alert,
  KeyboardAvoidingView,
  Platform
} from 'react-native';
import DateTimePicker from '@react-native-community/datetimepicker';
import { SafeAreaView } from 'react-native-safe-area-context';
import { AppContext } from '../context/AppContext';

export default function EntryScreen() {
  const { records, setRecords, sendToSheets } = useContext(AppContext);
  
  // Form state
  const [name, setName] = useState('');
  const [age, setAge] = useState('');
  const [phone, setPhone] = useState('');
  const [vehicle, setVehicle] = useState('');
  const [cargo, setCargo] = useState('');
  
  // DateTime state
  const [timeIn, setTimeIn] = useState(new Date());
  const [timeOut, setTimeOut] = useState(new Date(new Date().getTime() + 2 * 60 * 60 * 1000)); // +2 hours
  
  // Date pickers state
  const [showTimeInPicker, setShowTimeInPicker] = useState(false);
  const [showTimeOutPicker, setShowTimeOutPicker] = useState(false);
  
  // Loading state
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  // Generate ID
  const generateId = () => {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  };
  
  // Form validation
  const validateForm = () => {
    if (!name.trim()) {
      Alert.alert('Error', 'Driver name is required');
      return false;
    }
    if (!age.trim() || parseInt(age) < 18 || parseInt(age) > 100) {
      Alert.alert('Error', 'Age must be between 18 and 100');
      return false;
    }
    if (!phone.trim()) {
      Alert.alert('Error', 'Phone number is required');
      return false;
    }
    if (!vehicle.trim()) {
      Alert.alert('Error', 'Vehicle number is required');
      return false;
    }
    if (!cargo.trim() || parseInt(cargo) < 1) {
      Alert.alert('Error', 'Number of cargos must be at least 1');
      return false;
    }
    return true;
  };
  
  // Handle form submission
  const handleSubmit = async () => {
    if (!validateForm()) return;
    
    setIsSubmitting(true);
    
    const newRecord = {
      id: generateId(),
      name: name.trim(),
      age: parseInt(age),
      phone: phone.trim(),
      vehicle: vehicle.trim(),
      cargoCount: parseInt(cargo),
      timeIn: timeIn.toISOString(),
      timeOut: timeOut.toISOString(),
      entryTime: new Date().toISOString(),
      checkoutTime: null,
      status: 'Active'
    };
    
    // Add to local records
    const updatedRecords = [...records, newRecord];
    setRecords(updatedRecords);
    
    // Try to send to Google Sheets
    try {
      const result = await sendToSheets('addRecord', { record: newRecord });
      
      if (!result.success) {
        Alert.alert('Warning', 'Record saved locally but failed to sync with Google Sheets. Will try again later.');
      } else {
        Alert.alert('Success', 'Record added successfully and synced with Google Sheets.');
      }
      
      // Reset form
      resetForm();
    } catch (error) {
      Alert.alert('Error', 'Something went wrong. Record saved locally.');
    } finally {
      setIsSubmitting(false);
    }
  };
  
  // Reset form
  const resetForm = () => {
    setName('');
    setAge('');
    setPhone('');
    setVehicle('');
    setCargo('');
    
    const now = new Date();
    setTimeIn(now);
    setTimeOut(new Date(now.getTime() + 2 * 60 * 60 * 1000)); // +2 hours
  };
  
  // Date picker handlers
  const onChangeTimeIn = (event, selectedDate) => {
    const currentDate = selectedDate || timeIn;
    setShowTimeInPicker(false);
    setTimeIn(currentDate);
    
    // Ensure timeOut is after timeIn
    if (currentDate > timeOut) {
      setTimeOut(new Date(currentDate.getTime() + 2 * 60 * 60 * 1000)); // +2 hours
    }
  };
  
  const onChangeTimeOut = (event, selectedDate) => {
    setShowTimeOutPicker(false);
    if (selectedDate) {
      setTimeOut(selectedDate);
    }
  };
  
  return (
    <SafeAreaView style={styles.container}>
      <KeyboardAvoidingView 
        behavior={Platform.OS === "ios" ? "padding" : "height"}
        style={styles.keyboardView}
      >
        <ScrollView contentContainerStyle={styles.scrollContent}>
          <Text style={styles.title}>Cargo Entry Form</Text>
          
          <View style={styles.formGroup}>
            <Text style={styles.label}>Driver Name</Text>
            <TextInput
              style={styles.input}
              value={name}
              onChangeText={setName}
              placeholder="Enter driver name"
            />
          </View>
          
          <View style={styles.formGroup}>
            <Text style={styles.label}>Age</Text>
            <TextInput
              style={styles.input}
              value={age}
              onChangeText={setAge}
              placeholder="Enter age"
              keyboardType="numeric"
            />
          </View>
          
          <View style={styles.formGroup}>
            <Text style={styles.label}>Phone Number</Text>
            <TextInput
              style={styles.input}
              value={phone}
              onChangeText={setPhone}
              placeholder="Enter phone number"
              keyboardType="phone-pad"
            />
          </View>
          
          <View style={styles.formGroup}>
            <Text style={styles.label}>Vehicle Number</Text>
            <TextInput
              style={styles.input}
              value={vehicle}
              onChangeText={setVehicle}
              placeholder="Enter vehicle number"
            />
          </View>
          
          <View style={styles.formGroup}>
            <Text style={styles.label}>Number of Cargos</Text>
            <TextInput
              style={styles.input}
              value={cargo}
              onChangeText={setCargo}
              placeholder="Enter number of cargos"
              keyboardType="numeric"
            />
          </View>
          
          <View style={styles.formGroup}>
            <Text style={styles.label}>Time In</Text>
            <TouchableOpacity
              style={styles.dateButton}
              onPress={() => setShowTimeInPicker(true)}
            >
              <Text>{timeIn.toLocaleString()}</Text>
            </TouchableOpacity>
            {showTimeInPicker && (
              <DateTimePicker
                value={timeIn}
                mode="datetime"
                is24Hour={false}
                display="default"
                onChange={onChangeTimeIn}
              />
            )}
          </View>
          
          <View style={styles.formGroup}>
            <Text style={styles.label}>Expected Time Out</Text>
            <TouchableOpacity
              style={styles.dateButton}
              onPress={() => setShowTimeOutPicker(true)}
            >
              <Text>{timeOut.toLocaleString()}</Text>
            </TouchableOpacity>
            {showTimeOutPicker && (
              <DateTimePicker
                value={timeOut}
                mode="datetime"
                is24Hour={false}
                display="default"
                onChange={onChangeTimeOut}
                minimumDate={timeIn}
              />
            )}
          </View>
          
          <TouchableOpacity
            style={[styles.button, isSubmitting && styles.buttonDisabled]}
            onPress={handleSubmit}
            disabled={isSubmitting}
          >
            <Text style={styles.buttonText}>{isSubmitting ? 'Submitting...' : 'Submit Entry'}</Text>
          </TouchableOpacity>
        </ScrollView>
      </KeyboardAvoidingView>
    </SafeAreaView>
  );
}

// screens/RecordsScreen.js
import React, { useState, useContext, useEffect } from 'react';
import { 
  View, 
  Text, 
  StyleSheet, 
  FlatList, 
  TouchableOpacity, 
  Alert,
  TextInput,
  ActivityIndicator,
  RefreshControl,
  Modal
} from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { Ionicons } from '@expo/vector-icons';
import { AppContext } from '../context/AppContext';

export default function RecordsScreen() {
  const { records, setRecords, sendToSheets, config } = useContext(AppContext);
  
  const [filteredRecords, setFilteredRecords] = useState([]);
  const [searchQuery, setSearchQuery] = useState('');
  const [isSyncing, setIsSyncing] = useState(false);
  const [refreshing, setRefreshing] = useState(false);
  const [selectedRecord, setSelectedRecord] = useState(null);
  const [modalVisible, setModalVisible] = useState(false);
  
  // Filter records based on search query
  useEffect(() => {
    if (searchQuery.trim() === '') {
      setFilteredRecords([...records].sort((a, b) => new Date(b.entryTime) - new Date(a.entryTime)));
    } else {
      const query = searchQuery.toLowerCase().trim();
      const filtered = records.filter(record => {
        return record.name.toLowerCase().includes(query) || 
               record.vehicle.toLowerCase().includes(query) || 
               record.phone.includes(query);
      });
      setFilteredRecords([...filtered].sort((a, b) => new Date(b.entryTime) - new Date(a.entryTime)));
    }
  }, [records, searchQuery]);
  
  // Format date for display
  const formatDate = (dateStr) => {
    if (!dateStr) return "Not set";
    const dt = new Date(dateStr);
    return dt.toLocaleString();
  };
  
  // Sync records with Google Sheets
  const syncRecords = async () => {
    if (!config.scriptUrl) {
      Alert.alert(
        'Configuration Required', 
        'Please configure the Google Sheets URL in the Settings tab first.'
      );
      return;
    }
    
    setIsSyncing(true);
    
    try {
      const result = await sendToSheets('getAllRecords', {});
      
      if (result.success) {
        // Update local records from sheet data
        if (result.records && result.records.length > 0) {
          // Format the records from the sheet to match local format
          const sheetRecords = result.records.map(record => ({
            id: record.id,
            name: record.name,
            age: typeof record.age === 'string' ? parseInt(record.age) : record.age,
            phone: record.phone,
            vehicle: record.vehicle,
            cargoCount: typeof record.cargocount === 'string' ? parseInt(record.cargocount) : record.cargocount,
            timeIn: record.timein,
            timeOut: record.timeout,
            entryTime: record.entrytime,
            checkoutTime: record.checkouttime,
            status: record.status
          }));
          
          setRecords(sheetRecords);
          Alert.alert('Success', `Synced ${sheetRecords.length} records from Google Sheets.`);
        } else {
          Alert.alert('Sync Completed', 'No records found on Google Sheets.');
        }
      } else {
        Alert.alert('Sync Failed', result.error || 'Failed to fetch records from Google Sheets.');
      }
    } catch (error) {
      Alert.alert('Error', 'Something went wrong during sync.');
    } finally {
      setIsSyncing(false);
      setRefreshing(false);
    }
  };
  
  // Pull to refresh
  const onRefresh = () => {
    setRefreshing(true);
    syncRecords();
  };
  
  // Delete record
  const deleteRecord = async (id) => {
    Alert.alert(
      'Confirm Delete',
      'Are you sure you want to delete this record?',
      [
        {
          text: 'Cancel',
          style: 'cancel'
        },
        {
          text: 'Delete',
          style: 'destructive',
          onPress: async () => {
            // Update local records
            const updatedRecords = records.filter(record => record.id !== id);
            setRecords(updatedRecords);
            
            // Try to sync with Google Sheets
            if (config.scriptUrl) {
              try {
                await sendToSheets('deleteRecord', { id });
              } catch (error) {
                Alert.alert('Warning', 'Record deleted locally but may not be synced with Google Sheets.');
              }
            }
          }
        }
      ]
    );
  };
  
  // Checkout/complete record
  const checkoutRecord = async (id) => {
    const record = records.find(r => r.id === id);
    if (!record) return;
    
    Alert.alert(
      'Confirm Checkout',
      'Are you sure you want to complete this cargo delivery?',
      [
        {
          text: 'Cancel',
          style: 'cancel'
        },
        {
          text: 'Complete',
          onPress: async () => {
            // Update local record
            const updatedRecords = records.map(r => {
              if (r.id === id) {
                return {
                  ...r,
                  checkoutTime: new Date().toISOString(),
                  status: 'Completed'
                };
              }
              return r;
            });
            
            setRecords(updatedRecords);
            
            // Try to sync with Google Sheets
            if (config.scriptUrl) {
              const updatedRecord = updatedRecords.find(r => r.id === id);
              try {
                await sendToSheets('updateRecord', { record: updatedRecord });
              } catch (error) {
                Alert.alert('Warning', 'Record updated locally but may not be synced with Google Sheets.');
              }
            }
          }
        }
      ]
    );
  };
  
  // View record details
  const viewRecord = (record) => {
    setSelectedRecord(record);
    setModalVisible(true);
  };
  
  // Render list item
  const renderItem = ({ item }) => (
    <View style={styles.listItem}>
      <TouchableOpacity style={styles.itemContent} onPress={() => viewRecord(item)}>
        <View style={styles.mainInfo}>
          <Text style={styles.nameText}>{item.name}</Text>
          <Text style={styles.vehicleText}>Vehicle: {item.vehicle}</Text>
          <Text>Cargos: {item.cargoCount}</Text>
          <Text>Time In: {formatDate(item.timeIn)}</Text>
        </View>
        <View style={styles.statusContainer}>
          <Text style={[
            styles.statusLabel,
            item.status === 'Completed' ? styles.completedStatus : styles.activeStatus
          ]}>
            {item.status}
          </Text>
        </View>
      </TouchableOpacity>
      
      <View style={styles.actions}>
        {item.status !== 'Completed' && (
          <TouchableOpacity 
            style={[styles.actionButton, styles.checkoutButton]}
            onPress={() => checkoutRecord(item.id)}
          >
            <Text style={styles.actionButtonText}>Complete</Text>
          </TouchableOpacity>
        )}
        <TouchableOpacity 
          style={[styles.actionButton, styles.deleteButton]}
          onPress={() => deleteRecord(item.id)}
        >
          <Text style={styles.actionButtonText}>Delete</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
  
  // Empty list component
  const EmptyList = () => (
    <View style={styles.emptyContainer}>
      <Ionicons name="cube-outline" size={50} color="#ccc" />
      <Text style={styles.emptyText}>No cargo records found</Text>
      {searchQuery.trim() !== '' && (
        <Text style={styles.emptySubText}>Try a different search term</Text>
      )}
      {searchQuery.trim() === '' && records.length === 0 && (
        <TouchableOpacity
          style={styles.emptyButton}
          onPress={syncRecords}
        >
          <Text style={styles.emptyButtonText}>Sync with Google Sheets</Text>
        </TouchableOpacity>
      )}
    </View>
  );
  
  // Record details modal
  const RecordDetailsModal = () => (
    <Modal
      animationType="slide"
      transparent={true}
      visible={modalVisible}
      onRequestClose={() => setModalVisible(false)}
    >
      <View style={styles.modalOverlay}>
        <View style={styles.modalContent}>
          <View style={styles.modalHeader}>
            <Text style={styles.modalTitle}>Cargo Details</Text>
            <TouchableOpacity onPress={() => setModalVisible(false)}>
              <Ionicons name="close" size={24} color="#333" />
            </TouchableOpacity>
          </View>
          
          {selectedRecord && (
            <View style={styles.modalBody}>
              <View style={styles.detailRow}>
                <Text style={styles.detailLabel}>Driver Name:</Text>
                <Text style={styles.detailValue}>{selectedRecord.name}</Text>
              </View>
              <View style={styles.detailRow}>
                <Text style={styles.detailLabel}>Age:</Text>
                <Text style={styles.detailValue}>{selectedRecord.age}</Text>
              </View>
              <View style={styles.detailRow}>
                <Text style={styles.detailLabel}>Phone:</Text>
                <Text style={styles.detailValue}>{selectedRecord.phone}</Text>
              </View>
              <View style={styles.detailRow}>
                <Text style={styles.detailLabel}>Vehicle:</Text>
                <Text style={styles.detailValue}>{selectedRecord.vehicle}</Text>
              </View>
              <View style={styles.detailRow}>
                <Text style={styles.detailLabel}>Cargo Count:</Text>
                <Text style={styles.detailValue}>{selectedRecord.cargoCount}</Text>
              </View>
              <View style={styles.detailRow}>
                <Text style={styles.detailLabel}>Time In:</Text>
                <Text style={styles.detailValue}>{formatDate(selectedRecord.timeIn)}</Text>
              </View>
              <View style={styles.detailRow}>
                <Text style={styles.detailLabel}>Expected Out:</Text>
                <Text style={styles.detailValue}>{formatDate(selectedRecord.timeOut)}</Text>
              </View>
              <View style={styles.detailRow}>
                <Text style={styles.detailLabel}>Entry Time:</Text>
                <Text style={styles.detailValue}>{formatDate(selectedRecord.entryTime)}</Text>
              </View>
              {selectedRecord.checkoutTime && (
                <View style={styles.detailRow}>
                  <Text style={styles.detailLabel}>Checkout Time:</Text>
                  <Text style={styles.detailValue}>{formatDate(selectedRecord.checkoutTime)}</Text>
                </View>
              )}
              <View style={styles.detailRow}>
                <Text style={styles.detailLabel}>Status:</Text>
                <Text
                  style={[
                    styles.statusValue,
                    selectedRecord.status === 'Completed' ? styles.completedText : styles.activeText
                  ]}
                >
                  {selectedRecord.status}
                </Text>
              </View>
            </View>
          )}
        </View>
      </View>
    </Modal>
  );
  
  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Cargo Records</Text>
        
        <View style={styles.searchContainer}>
          <TextInput
            style={styles.searchInput}
            placeholder="Search by name, vehicle, phone..."
            value={searchQuery}
            onChangeText={setSearchQuery}
          />
          {searchQuery !== '' && (
            <TouchableOpacity 
              style={styles.clearButton}
              onPress={() => setSearchQuery('')}
            >
              <Ionicons name="close-circle" size={20} color="#777" />
            </TouchableOpacity>
          )}
        </View>
        
        <TouchableOpacity
          style={[styles.syncButton, isSyncing && styles.syncingButton]}
          onPress={syncRecords}
          disabled={isSyncing}
        >
          {isSyncing ? (
            <ActivityIndicator size="small" color="#fff" />
          ) : (
            <Text style={styles.syncButtonText}>Sync with Google Sheets</Text>
          )}
        </TouchableOpacity>
      </View>
      
      <FlatList
        data={filteredRecords}
        renderItem={renderItem}
        keyExtractor={item => item.id}
        contentContainerStyle={styles.list}
        ListEmptyComponent={EmptyList}
        refreshControl={
          <RefreshControl
            refreshing={refreshing}
            onRefresh={onRefresh}
            colors={['#3498db']}
          />
        }
      />
      
      <RecordDetailsModal />
    </SafeAreaView>
  );
}

// screens/SettingsScreen.js
import React, { useState, useContext } from 'react';
import { 
  View, 
  Text, 
  StyleSheet, 
  TextInput, 
  TouchableOpacity, 
  Switch,
  ScrollView,
  Alert,
  ActivityIndicator,
  Modal
} from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { AppContext } from '../context/AppContext';

export default function SettingsScreen() {
  const { config, setConfig, sendToSheets, errorLog } = useContext(AppContext);
  
  // Form state
  const [sheetId, setSheetId] = useState(config.sheetId);
  const [sheetName, setSheetName] = useState(config.sheetName);
  const [scriptUrl, setScriptUrl] = useState(config.scriptUrl);
  const [useCorsProxy, setUseCorsProxy] = useState(config.useCorsProxy);
  const [corsProxy, setCorsProxy] = useState(config.corsProxy);
  
  // Testing state
  const [isTesting, setIsTesting] = useState(false);
  const [testModalVisible, setTestModalVisible] = useState(false);
  const [testResults, setTestResults] = useState(null);
  const [showDebug, setShowDebug] = useState(false);
  
  // Save settings
  const saveSettings = () => {
    const newConfig = {
      sheetId: sheetId.trim(),
      sheetName: sheetName.trim() || 'Sheet1',
      scriptUrl: scriptUrl.trim(),
      useCorsProxy,
      corsProxy: corsProxy.trim()
    };
    
    setConfig(newConfig);
    Alert.alert('Success', 'Settings saved successfully.');
  };
  
  // Test connection
  const testConnection = async () => {
    const testConfig = {
      sheetId: sheetId.trim(),
      sheetName: sheetName.trim() || 'Sheet1',
      scriptUrl: scriptUrl.trim(),
      useCorsProxy,
      corsProxy: corsProxy.trim()
    };
    
    if (!testConfig.scriptUrl) {
      Alert.alert('Error', 'Please enter Google Apps Script URL first.');
      return;
    }
    
    setIsTesting(true);
    setTestModalVisible(true);
    setTestResults({
      status: 'testing',
      message: 'Testing connection...',
      details: {
        url: testConfig.scriptUrl,
        sheet: testConfig.sheetName
      }
    });
    
    try {
      // Use the local config for the test
      const originalConfig = { ...config };
      setConfig(testConfig);
      
      const result = await sendToSheets('testConnection', {});
      
      // Restore the original config if not saved
      setConfig(original
